---
layout: post
title: "I Was the Conductor. AI Was the Orchestra."
subtitle: "I wrote the score, not the code"
date: 2026-10-09
tags: [ai, claude, "product development", hcgi]
thumbnail-img: /assets/img/posts/act-launch-sq.jpg
cover-img: /assets/img/posts/act-launch-header.jpg
share-img: /assets/img/posts/act-launch-og.jpg
---

Claude did. All of it — every route, every migration, every test, every bug fix, every one of those 155 commits.

---
# I Was the Conductor. AI Was the Orchestra.

### I wrote the score, not the code

*This is the first post in a series on how ACT actually got built. This one's about the product and the working relationship — later posts will get into the infrastructure, the process and data workflows, and the tradeoffs I made along the way.*

For the last six weeks, I've been building [Awards Ceremony Tracker](https://awardsceremonytracker.com) — ACT — a tool that reads your [Letterboxd](https://letterboxd.com) diary and tells you what you've seen that's actually nominated, and what you're still missing, against the real Oscars, Golden Globes, and Independent Spirit Awards ballots. It launched this week — version 1.0.

I never touched the code. Every commit, every migration, every test came from [Claude](https://claude.ai). My job was the one a conductor actually has: know the piece cold, catch the wrong note before it reaches an audience, decide tempo and shape and what gets cut — and never pick up an instrument myself. That's not a cute framing I applied after the fact. It's close to literally how the six weeks went, and I want to walk through what that actually looked like in practice, because "I built an app with AI" has become a throwaway claim that undersells it.

## It started with a PRD, not a prompt

Before any code existed, I sat down with Claude — a separate conversation, not [Claude Code](https://claude.com/claude-code) — and worked through an actual Product Requirements Document. Not a one-line idea dumped into a chat window: a real PRD, with a defined problem statement, a data model, a milestone breakdown, open questions flagged explicitly as open rather than quietly assumed away. That document — `PRD.md` — is still in the repo today, still the thing every real product decision during the build got checked against.

That's the detail I think matters most about how this actually started. A conductor doesn't walk into rehearsal having never read the score — the PRD-drafting conversation was that read-through, and it was its own real work — defining what "nominated" even means across three different awards bodies, deciding up front that a canonical, body-agnostic category list was the right model instead of hardcoding per-body logic, scoping what v1 would and wouldn't attempt. By the time I handed that document to Claude Code as the actual spec to build against, the hard product thinking was already done. What followed for the next six weeks was building against a real plan, not improvising one feature at a time.

That's maybe the most under-discussed thing about building with AI tools right now: the planning conversation and the implementation conversation don't have to be — maybe shouldn't be — the same conversation. I used one tool to think, handed its output to another to build, and kept being the owner of both.

## The role I actually had

I was the technical product owner, which on this project meant the conductor's actual job, not the metaphor's soft version of it: I scoped every slice of work, made the architecture calls I had opinions on, reviewed every meaningful decision, pushed back when something felt wrong, and did the QA. Claude — specifically Claude Code, working through a long-running conversational session — wrote all the code, every time, for six weeks straight.

That split sounds clean in a sentence. In practice it was closer to how I'd work with a very capable, very fast engineer who has one weakness: no persistent memory of their own unless I build it for them. Which, in this project, I effectively did — more on that below.

A few examples of what "driving the product" actually looked like, not as abstractions but as real decisions:

- When a ranking I made under "Best Picture" was silently affecting every awards body that shared that category — a real bug I hit using my own app — I was the one who said "I want each of the potential categories to be distinct to the awards body," which became the actual spec for the fix.
- When Claude's first instinct for detecting a film's awards eligibility was a language/country heuristic, I corrected it with real examples — *Anatomy of a Fall* and *Sentimental Value* are both largely English-language films that competed internationally — and that correction became the actual rule the app ships with today.
- When I decided BAFTA wasn't worth tracking (no reliable data source, an actively hostile anti-bot wall on their own site), I said so directly, and it came out of the product entirely — code, docs, FAQ, the works.
- When I wanted the app's data reset before real users arrived, I asked for it, watched Claude's first attempt get blocked by its own safety guardrails, and then got offered — and accepted — a proper reviewable CLI tool instead of a workaround. That's a boundary I didn't know I wanted until it protected me from my own impatience.

None of that is "prompting." It's product ownership, exercised in real time, against a system that could actually execute at the speed I could make decisions.

## The reasoning isn't just in my commit history — it's in the product

Here's the part I didn't expect to end up liking: a lot of the decisions above aren't just things I remember making. They're answered, in plain language, in the app's own [FAQ](https://awardsceremonytracker.com/faq) — because a user hitting the same "wait, why doesn't this work the way I expected" moment I had deserves the same real answer I got, not a shrug.

A good example: mid-build, I caught a real bug — a $200 million blockbuster was showing up in the Independent Spirit Awards' "Best Feature" section, something that could never happen at the real ceremony. Not a spec I'd handed down up front — a call I made on the spot, once I saw it. Here's how the FAQ explains it today, verbatim:

> **Why does a film sometimes get its own separate Independent Spirit ranking for a category the Oscars also have?**
> Because the eligible field genuinely is different... Spirit's $30 million budget cap means its own field of contenders... is narrower. Rather than pretend one pooled ranking covers both, eligible films get both.

The fix isn't hidden in a changelog; it's a page any user can read right now.

Same thing with gender-split acting categories — three different bodies, three different conventions, no single rule forced across them. The FAQ just says so plainly:

> **Why are some acting categories split by gender and others aren't?**
> Because the real awards bodies don't agree, and this app follows each body's own actual categories rather than picking one convention for all of them. The Oscars still nominate Actor and Actress separately. The Golden Globes split further still... The Independent Spirit Awards use one gender-neutral Lead/Supporting Performance category. You'll see whichever shape of the category a tracked body actually uses.

Twelve FAQ entries exist today, and most of them trace back to a real moment where either I caught something wrong or a user would have. That's a different relationship to "why does this work this way" than most software ships with — the reasoning isn't proprietary, it's product copy.

## Who wrote the code

Claude did. All of it — every route, every migration, every test, every bug fix, every one of those 155 commits. My job was never to write [TypeScript](https://www.typescriptlang.org) — it was to know what *correct* looked like for this specific product, catch it when the implementation drifted from that, and make the calls only a human product owner can make (what to build next, what to cut, what risk is acceptable).

The thing that made this workable rather than chaotic was that Claude didn't just write code — it explained *why* at every real decision point, unprompted. Not "here's the code," but "here's the code, and here's the actual tradeoff I'm making and why I made it this way instead of the obvious alternative." That meant I could push back on reasoning, not just output. A lot of the real fixes in this project — the ones above included — came from me catching a wrong assumption *in the explanation*, before it ever became a shipped bug.

## How testing actually worked

This is the part I think gets skipped in most "I built an app with AI" posts, and it's the part I care most about getting right in this one, because it's where the whole approach could have fallen apart.

The short version: nothing shipped on the strength of "it looks right." Every real feature has real tests behind it — 432 of them as of today, split across three tiers:

- **132 unit tests** for pure logic — the stuff that doesn't touch a database or network, like the title-matching heuristics that resolve a Letterboxd watch against a [TMDB](https://www.themoviedb.org) film record.
- **265 integration tests that hit a real local [Postgres](https://www.postgresql.org) database**, not mocks. This was a deliberate, repeated choice. Early on, a mocked-database test passed clean while the real migration behind it was subtly broken — the kind of divergence that only shows up when the test actually touches the real thing. After that, the rule was simple: if it touches the database, the test touches a real database too.
- **35 frontend tests**, the newest tier, added specifically because I asked for test coverage on a UI change rather than trusting it by eye.

But the number that matters more than the count is the discipline behind it. A few real examples:

- When a real bug slipped into the code that checks whether someone's logged in before letting them see a page, a test built from a fake, hand-written stand-in for a real request couldn't catch it — only a test that actually ran against the real, live server could. That became a standing rule: anything that gates access to a page gets tested against the real thing, no exceptions.
- When a data-ingestion pipeline needed to pull real nomination data from TMDB, Claude didn't trust its own matching logic until it had run against *actual* messy real-world data and I'd confirmed the output by hand — not synthetic fixtures.
- When something genuinely couldn't be tested automatically — a visual regression, a touch-drag interaction on a real phone — that limitation got stated outright instead of glossed over. "I can't verify this one without a browser, flagging it" was a normal sentence in this project, not an admission of failure.

The philosophy that held it together was mine, and I said it out loud the one time it actually got tested: Claude suggested a manual QA pass as a safety net, and I pushed back — the whole point of automated testing and deployment is that when code ships, it *works*, not that a human has to go double-check it by hand afterward. A system that lies about success gets fixed at the source, not patched over with a standing manual-check ritual. We hit that lesson for real: a critical setup step silently never ran in production — for the entire history of the project — because of one small configuration mistake, and every deploy still reported "success" the whole time. The fix wasn't "remember to check by hand next time" — it was a new safeguard that fails loudly, automatically, if that ever happens again.

## What surprised me

The thing I didn't expect going in: the slowest part of this project was never the code. It was the decisions only I could make — what a "correct" ranking means when two award bodies disagree about a category, whether a punctuation mismatch in a film title is worth a general fix or a one-off correction, when "good enough to ship" is actually good enough. Claude could produce a working implementation of almost anything I specified within minutes. It could not decide, on its own, what this product was actually *for*. That part stayed entirely mine, for the whole six weeks.

ACT is live now at [awardsceremonytracker.com](https://awardsceremonytracker.com). If you're the kind of person who tracks a Letterboxd diary and still loses the thread on what's nominated this year, that's exactly who it's for.

## This is launch day, not finished

I want to be straight about something, because I think the honest version of this story matters more than the triumphant one: ACT being live doesn't mean ACT is done, or that I'm fully confident in every corner of it yet. What Claude and I built together is real, it's tested, and it genuinely works — but six weeks of rigorous testing is not the same thing as software that's absorbed real usage from real people with real, messy Letterboxd histories going back a decade or more. I already expect to be finding bugs myself over the next few weeks — in features, and in the underlying data (a title that doesn't match, a nomination that's missing, a ranking that behaves oddly) — the kind of thing that only surfaces once actual humans start actually using it.

There's a report-a-problem link built into the app for exactly this reason. See something wrong? Tell me. That's how this actually gets from an AI-developed prototype to something rock-solid — not me guessing at edge cases alone, but real cinephiles on Letterboxd flagging what's broken. If that's you, consider that link an invitation to help build this, not just use it.

## Coming up in this series

This post was about the product and the working relationship. Still to come:

- **The infrastructure** — how ACT is actually hosted, deployed, and kept running, and the real incidents that shaped those decisions
- **The process and data workflows** — how awards and film data actually gets into the app, and what it looks like to catch and fix data problems at scale
- **The tradeoffs** — what I'd do differently, what this approach is genuinely bad at, and where I overrode the AI's first instinct and why

## If you're sizing up this approach for your own project

This is the workflow now, for anything I build: I own the product, Claude owns the implementation, and I do the kind of real review that catches a wrong assumption before it ships. ACT is the proof it holds up past a weekend prototype — six weeks, a real launch, a real test suite, real users.

If you're a founder or a product lead trying to figure out whether this approach is right for what you're building — or you're already building this way and want a second set of eyes on how it's actually going — that's the conversation I like having. [Reach out](mailto:robbie@hcgi.io) and let's talk about your project.

---

## The numbers, for anyone who wants the receipts

- **Timeline:** 43 days from the first day of work to launch — 22 of those were active build days
- **Scope of work:** 155 individual changes to the application (commits), grouped into 153 reviewed and shipped feature/fix batches (pull requests)
- **Finished codebase:** roughly 31,400 lines of code
