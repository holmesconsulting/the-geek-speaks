---
layout: post
title: "Every Player Had to Audition First."
subtitle: "ACT's Data, every feature was driven by the data, it was tested and verified before it was trusted and relied on."
date: 2026-11-06
tags: [ai, claude, "product development", hcgi, data]
thumbnail-img: /assets/img/posts/act-data-sq.jpg
cover-img: /assets/img/posts/act-data-header.jpg
share-img: /assets/img/posts/act-data-og.jpg
excerpt: "Pulling a user's Letterboxd diary is where it started then things got complicated. Data was the driving force of ACT's features no matter what the product requirements document said, or I wanted."
---
*This post is Part 3 in the ACT development series on [The Geek Speaks](https://thegeekspeaks.io).*

- Part 1: ["I Was the Conductor. AI Was the Orchestra."](https://thegeekspeaks.io/lauch-of-act/)
- Part 2: ["The Stage Was Built Before the First Note."](https://thegeekspeaks.io/act-the-infrastructure/)



I started the process assuming I'd pull diary data from [Letterboxd](https://letterboxd.com)'s RSS feed —
there's no usable Letterboxd API, so RSS plus Letterboxd's own data export for full
history was always the plan. I knew the feed held 50 entries; what I didn't know was
that 50 was a hard cap with no pagination, no way to reach anything past it. The
other open question was how to match an export that carries no film id at all back
to a real record. That same habit — not assuming, checking — is what later decided
which four award ceremonies the app actually tracks.

## No API, and that was fine.

Letterboxd does have a documented API for pulling a member's own diary, but access
is request-only, and the category this project falls into — data analysis, LLMs,
personal projects — is explicitly declined, not just slow to approve. That was known
going in. It's why RSS plus the data export was the plan from day one, not a
fallback forced by surprise.

## What the RSS feed will and won't do.

Documented limits aren't always the real limit. Feeds and APIs often have an
undocumented way around a stated cap, so before accepting Letterboxd's RSS feed at
50 entries, I ran it through a few of the known patterns for getting past that kind
of wall — a page parameter being the obvious first try. Diffed the bytes between
what page one and page two actually returned. Nothing moved. They came back
identical: there's no path past 50, and that's a tested conclusion, not a guess.

Lucky break: Letterboxd's own data export already existed — not built for this, but
it was the one place a full history could still come from. That's what got dug into
next.

The export turned out to be a Pro/Patron-only ZIP — `watched.csv` for full coverage,
`diary.csv` for the dated subset. Neither one carries a [TMDB](https://www.themoviedb.org)
or [IMDb](https://www.imdb.com) id. Just a Letterboxd URL. So matching several
thousand rows of it back to a real film became its own real problem, not a
formality.

TMDB — The Movie Database — is the one piece of this pipeline that didn't need
special handling. It has a real, public API built for exactly this kind of project,
and its own limits are generous and, by design, unpublished — nothing like
Letterboxd's request-only, explicitly-declined policy. That's what made it the
first thing tried for every title in the export, not a reluctant fallback.

## Being a polite guest, not a scraper.

A first full-history import means resolving a few thousand titles at once, and
nothing about that process was allowed to look like an attack on Letterboxd's own
servers. The design: TMDB search tried first for every title, a direct scrape of
Letterboxd's own page only as a last resort, one request at a time, a couple of
seconds apart, an honest user-agent, robots.txt respected.

That last resort — the scrape fallback — never got built at all. The reasoning
written into the code at the time still holds: it's real complexity, worth adding
once it's clear it's actually needed, not before. Every title TMDB's own search
can't resolve on its own just surfaces for a person to match by hand instead.

## Hydration — once per film, not once per person.

Matching answers "which film is this." Hydration is what happens right after: TMDB
is the real record here — Letterboxd's own film database sits on top of it, and a
diary export hands over a title, a year, sometimes a link, nothing like the cast,
crew, budget, or poster ACT actually needs to run categorization and awards
matching. So hydration pulls all of that in once a film is matched, and stores it
locally.

It only happens once per film, not once per person. The first diary entry that
points at a given film triggers the real fetch; everyone else who's watched it,
now or years from now, reuses what's already stored. Crew gets filtered on the way
in too — TMDB can report well over a hundred job titles for one film's crew, and
only the handful that actually feed an awards category are worth keeping.

Budget got its own real workaround, because it's not just metadata here — it's the
number [Independent Spirit](https://www.filmindependent.org/spirit-awards/)'s own $30 million eligibility cap runs on. And TMDB's
budget field goes missing most often for exactly the films that cap cares about:
newly released ones, where real-world reporting hasn't caught up yet.

That gap showed up for real on *The Social Reckoning* — a big-budget studio sequel
with no TMDB budget figure at all. Without a figure, it would have looked
Spirit-eligible, not because it was actually under the cap, but because the number
simply wasn't there.

The fix had to go the other way from the obvious instinct. A true indie film is the
one more likely to be missing a TMDB budget figure, not less — so a missing number
defaults to eligible, not ineligible. Penalizing real indie films for a data gap
would be the worse failure, not letting one studio sequel occasionally need a
second look.

That second look: Wikidata first, for a structured cost figure tied to the same
film. If that comes up empty but the film has a linked Wikipedia page, the app
parses that page's own infobox directly out of the raw wikitext. When the reported
figure is a range, it takes the high end on purpose — the job is catching
over-cap films, not landing on a precise number.

[More on how the cap itself narrows categories →](https://awardsceremonytracker.com/faq#spirit-budget-cap)

The MVP version of this was about as simple as it gets: look up a film, insert it,
move to the next one — one row at a time, however many thousand times a full
history needed. That stayed in place on purpose. There was already a backlog ticket
for batching it, already researched, just not prioritized — because there was no
real load yet to justify it.

Then there was real load: my own first full-history import, a few thousand titles,
each one doing roughly eighty sequential round-trips to the database for cast and
crew alone. The same [Sentry](https://sentry.io) setup from last post caught this inefficiency, surfaced
in the one [Slack](https://slack.com) channel every automated failure already posts to — not a support
ticket. That was the signal to pivot: the backlog ticket got pulled forward and
prioritized, now that the need was real instead of theoretical. The same pass
turned up new imports resolving one row at a time instead of five in parallel,
fixed alongside it. Neither one got solved ahead of the problem; both got fixed the
moment the need became real, not before.

## The same test, turned on the ceremonies themselves.

None of that — the API refusal, the byte-diffed RSS limit, the hydration bug that
only showed up under real load — got decided by a spec. It got decided by actually
checking. That same habit, pointed next at ACT's list of tracked ceremonies, is why
the four that shipped aren't the same four the original plan had in mind.

## Oscars, Golden Globes, and Actor Awards — confirmed, in.

The [Oscars](https://www.oscars.org) had the easiest case: a [GitHub dataset](https://github.com/DLu/oscar_data) that's actively maintained and
current through this year's ceremony became the backbone, cross-checked against a
second dataset and a live API for whatever season is still in progress. Both checked
out against real results before either one got trusted.

[Golden Globes](https://goldenglobes.com) almost ran on [Wikidata](https://www.wikidata.org) by default — it was the fallback plan. Then it
turned out the Globes' own website quietly exposes its own data feed, undocumented,
used by nobody but its own site. Pulled a real ceremony's actual winner against it
before swapping the plan.

[Actor Awards](https://actorawards.org) wasn't part of the original four. Partway through, it was obvious there
was a real gap: a major, televised awards body that genuinely matters to awards
season, and a plausible reason someone would come to this app specifically to track
it. That's a product call, not a data call — but it only turned into a fourth tracked
ceremony because the data held up to the same bar as everything already in the app.
If it hadn't, the list would still be three, no matter how good the use case was.

## BAFTA was in the plan from day one. It's not in the app.

The original product spec had four ceremonies, and [BAFTA](https://www.bafta.org) was one of them — not a
stretch goal, a planned body from the start. It's the one that got cut.

The investigation wasn't shallow. There's no maintained dataset anywhere for it,
unlike the Oscars. Its own website sits behind real anti-bot infrastructure — not a
polite `robots.txt` request to back off, but the kind of active defense built to stop
even a browser pretending to be a person, let alone a script. And the fallback plan,
Wikidata, turned out to be worse than it looked: the most recent BAFTA Best Film
winner on record there was already two ceremonies out of date, and its nominee
history covers a fraction of what it should.

So BAFTA's gone — not paused, not "later." Every reference to it came out of the app:
the tracked-body list, the UI labels, even the FAQ line that used to explain why it
wasn't there yet.

But the door isn't sealed, it's just currently missing its one required ingredient.
Every source — a dataset, an API, a scraped page — gets wrapped in the same adapter
shape, so the ingestion core never changes just because a new body gets added. That
part's already built, and so is the research: we know exactly what's been tried and
ruled out for BAFTA specifically. If a real, trustworthy source ever shows up, adding
it back isn't a redesign — it's a feature branch.

## Independent Spirit got written off, then came back.

This one mattered to me personally — I'm a [Film Independent](https://www.filmindependent.org) voter, so Spirit was
never going to be an afterthought. The first real look at its data wasn't
encouraging: every automated option had a real problem, and the conclusion was to
fall back to parsing [Wikipedia](https://www.wikipedia.org)'s raw wikitext plus manual entry for whatever that
missed.

I pushed back on that. Not because I had a better source in mind — just: is there
really no way to pull this automatically? That question is what sent it back for a
second, deeper look.

The second look split the difference. Film Independent's own nominee hub page really
is dead — no real content without running its JavaScript, same conclusion as before.
But every category's own page underneath it isn't. Each one is plain, ordinary HTML,
sitting there unprotected, never checked directly the first time because the hub page
above it looked bad enough to stop looking further.

Scraping those pages only gets a nominee's name as text. Resolving that name to a
real film or person in the database is its own step — the same shape of problem as
the Letterboxd export, just on the ingestion side instead of the user side. Season
2025's first real run: 68 of 70 resolved automatically with confidence, 95.7%,
across all 12 categories. The 2 that didn't resolve automatically got surfaced as misses, not silently dropped — one because TMDB doesn't have that title yet, one because TMDB doesn't list that crew credit. Claude and I dug in manually, right then, the same way as everything else in this post that needed a second look — not left waiting on TMDB to eventually catch up.



That one reversal is the whole thesis of this post in miniature: the same source can
get a "no" and a "yes" depending on how far you actually look, and the right answer to
"can we trust this" is never the first verdict, it's the one that survives someone
asking "are you sure?"

This is the human-in-the-loop part of the whole series, not just this post: a gut
instinct — is there really no way? — doesn't come from a spec or a test suite, it
comes from someone who knows the subject pushing back. Sometimes that
reinvestigation lands exactly where the first pass did. Sometimes, like here, it
adds an entire awards body back into the product. Either way, it only works at this
pace because an AI partner can run far more research cycles, far faster, than I
could alone.

## Three different kinds of wrong, three different fixes.

Worth naming here: ACT has three tiers, not one undifferentiated "user." Everyone
gets the basics — their own diary, rankings, reporting a problem. Power users get
more: direct edit access to a film's own data, plus early access to features still
rolling out. And, of course, there's an admin tier above that, with full account
management.

Matching and hydration aren't guaranteed to land clean every time, and "wrong"
doesn't always mean the same thing, so there are three separate ways to correct it.

When matching itself can't land a confident result, that title doesn't just
disappear. It shows up on its own page with a real search built in, so the user can
find and pick the right film themselves. If it genuinely isn't a match, dismissing
it isn't just a delete. It asks which of a handful of known reasons applies. That
answer gets kept, not thrown away. So a pattern worth fixing properly gets noticed,
instead of quietly piling up unseen.

When the match was right but something hydration pulled in — a release year, a
budget figure — turns out wrong at the source, that's a different tool: a power
user can correct a film's own data directly, with a full history of revisions and
the ability to revert any edit.

And for everything else — anything a regular user notices that's wrong, anywhere in
the app, that doesn't fit either of those two shapes — there's one field: say what's
wrong, nothing else to fill in. Who's reporting, what page, which film — all captured
automatically, and it opens as a real, triaged GitHub issue, not a message that
disappears into an inbox.

## What I'd tell you to plan for.

The original plan and the shipped product don't match. The gap comes down to two
questions, asked of every ceremony's nominee and winner data: is this accurate right
now, and will it still be updated next season and the one after?

Checked Wikidata's BAFTA record against a real ceremony result, and it was already
two cycles out of date — that's what ruled the fallback out, not a hunch. The scrape
option didn't hold up either: even where something could be pulled once, there was
no reliable way to keep pulling it every season. Independent Spirit didn't pass
either check the first time I looked — it took a second, deeper look to get there.
Golden Globes and Actor Awards were straightforward: both checked out immediately,
straight from each body's own site.

Four ceremonies shipped — not because I picked them, because the data did. Every
one of them earned its spot by surviving a real check, not by being on the
original plan.

## How any of this gets trusted before it ships.

None of the matching, hydration, or correction logic above shipped on faith. It runs
through three different kinds of checking, and each one catches something the other
two can't.

Automated tests cover the logic itself — every adapter, every matching heuristic,
every correction path has its own suite, run against a real database, not mocked
responses standing in for one. That catches regressions and edge cases you can name
in advance.

Then there's a power-user flag, and I was the first person it ever got turned on
for. A feature can ship to production and sit behind that flag before anyone else
sees it — so the first real use of it, against real data, is mine, not a synthetic
fixture standing in for one.

And under all of that is plain manual QA: actually using the app the way a real
user would, with a real Letterboxd history, real searches, real imports. That's the
layer the other two can't replace — automated tests only check what someone already
thought to test, and a power-user flag only helps if someone actually uses the
feature it's gating.

## Coming up.

Next: the tradeoffs. Matching and hydration don't always get it right the first
time. Some of the real misses are worth telling honestly — what broke, and why.
That post covers the actual bugs, what I'd do differently, and the monitoring that
got built because of them — the real mechanics for watching this pipeline in
production, not guessing at how it's doing after the fact.

## The receipts.

- **4** ceremonies tracked
- **1** planned, dropped for real — BAFTA
- **~80 → batched** sequential TMDB round-trips per film, found via my own first full-history import
- **7,031** films hydrated from TMDB
- **118,609** people (cast + crew) pulled in along with them
- **68 / 70 (95.7%)** Spirit nominees auto-matched to a real film or person, season 2025
