---
layout: post
title: "The Stage Was Built Before the First Note."
subtitle: "ACT's infrastructure: CI/CD from commit one, and what changed between the plan and launch"
date: 2026-10-13
tags: [ai, claude, "product development", hcgi, infrastructure]
thumbnail-img: /assets/img/posts/act-part2-sq.jpg
cover-img: /assets/img/posts/act-part2-header.jpg
share-img: /assets/img/posts/act-part2-og.jpg
excerpt: "ACT never had a launch-day deploy. By the time it went live, every merged change had already been shipping itself to production. Here’s the pipeline behind that, and the three parts of the plan that didn’t survive contact with production."
---

*Part 2 of a series on **building [ACT, the Awards Ceremony Tracker](https://awardsceremonytracker.com)**. [Part 1](https://thegeekspeaks.io/lauch-of-act/) covered the PRD, my role as conductor, and testing. Next up, as promised: infrastructure.*

ACT never had a launch-day deploy. By October 9 it had already shipped to production well over a hundred times, because from the very first commit, every merged change deployed itself. That pipeline is the real infrastructure story. It's also why the plan we wrote in August could change safely on the way to launch, which is the other half of this post.

## The plan on paper

The product requirements document (PRD) set three rules for infrastructure. Be deployable as one unit: a single [Node](https://nodejs.org) service plus one managed [Postgres](https://www.postgresql.org) database. Never commit a secret. Get a trivial version live and boring early, then ship every milestone to production as it lands.

That last rule shaped everything else. Milestone 1 wasn't a feature. It was deploying an empty app to the internet. By the time ACT could do anything useful, the path to production had already been walked many times.

## The pipeline, from commit one

Every change takes the same path:

1. One small slice of work on its own branch.
2. I QA it.
3. A pull request, where CI runs lint, formatting, type checks, the test suite (including integration tests against a real Postgres), and a production build.
4. Branch protection on `main` blocks the merge until all of it passes.
5. Squash-merge, and [GitHub Actions](https://github.com/features/actions) deploys to [Fly](https://fly.io). Database migrations run as part of the deploy. Nobody touches the production database by hand.

The deploy job runs the entire verify suite a second time before shipping. That's redundant on purpose. If I ever bypass branch protection as an admin, which is exactly what you do when you're tired and in a hurry, broken code still can't reach production. The guardrails aren't there because I don't trust [Claude](https://claude.com/claude-code). They're there because I don't fully trust myself in a hurry, either.

That's what building the stage first buys you. Nothing waits for launch day, so nothing piles up for it.

## Side by side

|  | Planned (August) | Shipped (October) |
| --- | --- | --- |
| Shape | One Node service + one managed Postgres | Same |
| Hosting | [Fly.io](https://fly.io), scale to zero | Same, kept warm through launch |
| Database | [Neon](https://neon.com) (serverless Postgres) | Same, still on the free plan |
| Deploys | GitHub Actions, deploy on merge | Same, plus a second full test run before every deploy |
| Database access layer | [Prisma](https://www.prisma.io) | [Drizzle](https://orm.drizzle.team) |
| Login | Hand-built OAuth flow, as a learning exercise | Maintained [Fastify](https://fastify.dev) OAuth and session libraries |
| Background jobs | A timer inside the app | An hourly GitHub Actions workflow |
| Watching it | "Structured logs" | Error tracking, log search, uptime checks, [Slack](https://slack.com) alerts, daily backups, a cost watchdog |

The top half held. The bottom half is where the story is.

## Three changes, three reasons

**The timer that would have lied.** Scale to zero means the server stops when nobody is using it. A timer inside a stopped server doesn't fire, and it doesn't throw an error either. The diary sync would have run whenever someone happened to visit and silently skipped the rest of the time. So the schedule moved outside the app: an hourly GitHub Actions workflow calls a few protected endpoints, which wakes the machine the same way a visitor would. When I floated adding a real job queue, the answer was that nothing here is high volume. A scheduled workflow was the whole answer.

**The [Docker](https://www.docker.com) image that picked our ORM.** Prisma was the first choice and lasted until Milestone 2. Its code-generation step and native binary kept dragging tooling into the production image, which ballooned. That friction would have come back with every schema change. We switched to Drizzle before any real schema existed, which made it cheap. Six milestones later, it wouldn't have been.

**The learning exercise I skipped.** The plan was to hand-build the login flow for the first OAuth provider, just to understand it, and use a community library for the second. That made sense when I expected to be the one at the keyboard. It stopped making sense when I moved from hands-on pairing to technical product owner (more on that shift in the Tradeoffs post). Hand-rolling the first provider would have meant writing it once, adopting the library for the second, and then migrating the first one onto that library anyway. That's three passes at the most security-sensitive code in the app. I went straight to the stable, community-supported solution instead.

## The row no plan predicted

The biggest gap in that table is the last row. The plan said "structured logs." What shipped is a whole layer for watching production: error tracking in [Sentry](https://sentry.io), searchable logs in [Axiom](https://axiom.co), an uptime check every five minutes, daily database backups, and a job that checks every six hours for anything that would quietly run up the bill. Every automated failure posts to one Slack channel.

None of it was in the plan. It got added once production was real and the question changed from "does it work?" to "how would I know if it stopped?"

The clearest example: every deploy has two jobs to do before the new version goes live. First, update the structure of the database. Second, load ACT's reference data, the hand-curated list of awards bodies and categories. For a while, only the first job was happening. Every deploy still reported success. The second step was being skipped because of how the instructions were written, and nothing said a word about it. Production just quietly went without data it was supposed to have.

Picture a stagehand who sets the lights, never sets the props, and still signs off on the checklist. Tests couldn't have caught it, because tests don't run the real deploy. Only production could, and only because production existed early and something was watching. The fix made each step speak up when it fails. It also changed how I read a green checkmark: a deploy that says it worked is a claim, not proof.

## What I'd tell you to plan for

The architecture held because it was boring on purpose: one service, one database, mainstream tools. What changed were the details that only show up in production. That's the argument for putting production in week one. You can't plan for those details, but you can find them early, while they're still cheap to fix.

And budget for the watching layer from the start. Your AI pair will happily build you a deploy pipeline. It won't decide on its own that a green checkmark isn't proof. That part is still the conductor's job.

## Coming up

- **Data workflows.** How awards and film data get into ACT, why BAFTA didn't make it, and how data problems get found and fixed.
- **Tradeoffs.** What I'd change, where this approach struggles, and where I overrode the AI's instincts.

## The receipts

- Production releases: 165 on Fly, 151 of them deployed automatically on merge (the rest are configuration and secret changes)
- Full test-suite runs: 401 (250 on pull requests, plus 151 more right before each deploy), with 432 tests in the suite at launch
- Deployable units: 1
- Production machines: 1 small Fly machine, about $4.40/month while kept warm
- Database: Neon, free plan
- Planned infrastructure choices changed before launch: 3 (data access layer, login, background jobs)
- Hand edits to the production database: 0

*Counts are successful GitHub Actions runs as of October 13, 2026.*
