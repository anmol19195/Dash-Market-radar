# Building a Market Radar from scratch — reusable prompt kit

This is the reverse-engineered recipe for the DASH Market Radar, rewritten so it
works for any industry. It is four prompts run in sequence in a single Cowork
session, plus a config block you fill in first.

The whole thing rests on one idea worth stating up front: **a competitive radar
is not a research report with a nice skin on it.** It is a small database with
an opinion. The research is the cheap part. What makes it useful six months
later is that every entity has the same fields, every claim has a source, and
the file is small enough to regenerate in an afternoon.

---

## How the kit works

| Prompt | What it does | Roughly |
|---|---|---|
| 0 — Brief | You fill in a config block. Nothing runs. | 10 min of your thinking |
| 1 — Research | Fans out across the market, returns a structured research pack | 20–40 min |
| 2 — Build | Turns the pack into the single-file dashboard | 30–60 min |
| 3 — Ship | GitHub repo, Vercel, scheduled auto-refresh | 15 min |

Run them in one session so context carries. If you split them, paste the
research pack back in at the start of Prompt 2.

---

# PROMPT 0 — The Brief

Not a prompt. Fill this in first, in a scratch file. Prompts 1 and 2 both
reference it, and the quality of your answers here sets the ceiling for
everything downstream. Be specific; vagueness in this block becomes
generic filler in the dashboard.

```
RADAR BRIEF
===========
INDUSTRY:            [e.g. "footwear", "B2B payroll software", "specialty coffee retail"]
GEOGRAPHY:           [primary market + secondary, e.g. "India primary, global context"]
WHO IS THIS FOR:     [e.g. "founder + 3-person leadership team, read on phone, weekly"]
DECISION IT SERVES:  [the actual question, e.g. "where do we position, and who
                      do we need to beat to the narrative?" — not "understand the market"]

US
--
COMPANY:             [name, one line on what it is]
WEDGE:               [the one thing you do that others structurally cannot]
NOT-US:              [what you must never be described as — the framing you reject]
STAGE:               [pre-launch / launched / scaling — sets how honest the risks section is]
ASSETS:              [proprietary data, manufacturing, partnerships, distribution]
PARTNERS:            [who is an ally, so the radar never files them as a competitor]

THE FIELD
---------
INCUMBENTS:          [who owns the category today — 4-8 names]
CHALLENGERS:         [who is taking share — 6-12 names]
DISRUPTORS:          [the tech or model that could reset the rules — 3-8 names]
ENABLERS:            [infra, suppliers, tooling everyone depends on — 3-6 names]
CLOSEST ANALOGUE:    [the one company most structurally similar to you, anywhere]

SCOPE RULES
-----------
IN SCOPE:            [what belongs]
OUT OF SCOPE:        [what must be excluded, and why — be explicit; this is the
                      rule that stops the radar bloating over time]

BRAND
-----
PALETTE:             [3-5 hex values, or "neutral"]
TYPE:                [display font / body font / mono font, or "system"]
TONE:                [e.g. "flat, factual, slightly blunt. No hype adjectives."]
REFRESH CADENCE:     [e.g. "every 2 days", "weekly", "monthly"]
```

**On OUT OF SCOPE.** This is the field people skip and regret. On the footwear
radar it reads *"companies whose business is custom insoles — this is a
footwear radar"*. Without it, every refresh cycle drags in adjacent players
until the thing is about nothing in particular.

---

# PROMPT 1 — Research

Paste your filled-in brief, then this.

```
Research the market described in the brief above and produce a structured
research pack. Do not build anything yet — this phase is pure gathering.

Fan out across these seven organs. Every competitive radar has them; the
names change by industry, the function does not:

1. THE MAP — market size, growth rate, segmentation, and where value is
   concentrated. Give a range with sources rather than one confident number;
   research houses disagree and the disagreement is itself informative.

2. THE INCUMBENTS — who owns the category now. For public companies pull
   the last reported revenue, growth, profit, market cap and fiscal year.
   For private ones get what is disclosed and say when nothing is.

3. THE CHALLENGERS — who is taking share. Founding year, funding history
   with lead investors, latest disclosed revenue, price positioning, and
   the one-line reason they exist.

4. THE DISRUPTORS — the technology or business model that could reset the
   rules. What ships today versus what is a demo. Unit costs and cycle times
   where findable; these are what separate real from theatrical.

5. THE ENABLERS — infrastructure, suppliers, tooling. Often the highest-signal
   and least-covered layer, because consolidation here reshapes everyone above it.

6. THE MONEY — funding rounds, M&A, IPOs, take-privates, major exec moves,
   with dates. This becomes a timeline, so dates are mandatory.

7. REGULATION AND MACRO — anything that changes cost structure or market
   access: tariffs, quality standards, tax, subsidy programmes, trade policy.

For each entity return a consistent record:
  name · category · one-line position · go-to-market approach ·
  latest disclosed numbers with fiscal year · most recent notable move ·
  source name + URL

RULES
- Every figure traces to a citable source. Name it. If you cannot confirm
  something, either omit it or label it unconfirmed — never smooth over a gap.
- Prefer primary sources: investor relations, filings, company newsrooms.
  Trade press second. Aggregators and AI summaries last, and flagged.
- Carry the fiscal year with every financial figure. "FY26 (Mar)" not "2026".
- Where research houses disagree on market size, report the range and say so.
- Note what you could NOT find. Absence of coverage is a finding — it often
  marks exactly the whitespace the radar exists to identify.

Return the pack as structured text grouped by the seven organs. No prose
introduction, no dashboard yet.
```

**Worth knowing:** this phase parallelises well. If the session supports
subagents, ask for one per organ and it finishes in a third of the time. The
research pack is also worth keeping as a separate file — it is the audit trail
behind every number in the finished dashboard.

---

# PROMPT 2 — Build

The long one. This is the actual specification.

```
Using the brief and the research pack, build a single self-contained HTML
dashboard. Everything inline — CSS, JavaScript, data. No build step, no
framework, no local storage. One file that opens anywhere.

ARCHITECTURE
------------
- Tabbed single page. A sticky nav row switches sections by toggling a
  .tab class. No routing, no page loads.
- All content lives in JavaScript arrays near the bottom of the file,
  rendered into the DOM on load. Never hand-write content into HTML —
  data and presentation stay separate so refreshes only touch the arrays.
- One array per entity type, each with a consistent field shape. This is
  the single most important constraint in the build: it is what makes the
  thing maintainable six months out.
- Charts via Chart.js from a CDN with an integrity hash. Fonts from Google
  Fonts. Nothing else external.

SECTIONS
--------
Build these, in this order. Rename them to the industry's own language:

  Overview        The map. Headline stat tiles, market-size chart, and the
                  five forces shaping the category — each stated as a fact
                  followed by what it signals for us.
  [Home market]   Incumbents with financials, plus the challenger cohort
                  as a sortable, searchable table.
  [Global/wider]  The majors, filterable by sub-category, with a revenue
                  comparison chart.
  [Disruptors]    The technology or model that resets the rules. Split into
                  platforms, big-player plays, and enablers.
  Market Intel    What analysts and research houses actually say, with the
                  aggregate view and — more usefully — where they disagree.
  Watchlist       Ranked by threat level, high to low. Each entry: why they
                  matter, and specifically what to watch next. This is the
                  section people actually return to.
  Us vs Field     Honest comparison by competitor cluster: what we win on,
                  what they win on, and a verdict line per cluster. Then a
                  short list of real edges and a shorter list of real risks.
                  If the risks section is comfortable to read, it is wrong.
  Price–Value     Bubble chart positioning everyone on price against
                  perceived value, with us plotted on it.
  Channels        Distribution matrix — brands down, channels across,
                  each cell scored core / active / limited.
  GTM Tracker     Per-brand go-to-market detail. Clicking a brand opens
                  its full strategy plus a story timeline assembled
                  automatically from every dated mention of it elsewhere
                  in the data.
  Playbook        Marketing and launch tactics by brand: events, cadence,
                  tactics. The section you raid when planning a launch.
  Social Signals  Handles, follower counts, and content style per brand.
  Community       The cultural infrastructure — clubs, retailers,
                  tastemakers — and what each is worth to us.
  Money & Moves   Filterable timeline: funding, M&A, IPO, strategic.
                  Newest first.
  What's New      Rolling feed of recent developments. The section the
                  scheduled refresh rewrites.

Drop any that do not apply and add industry-specific ones. Fifteen is the
upper bound before it stops being scannable.

DESIGN SYSTEM
-------------
- Define the full palette as CSS custom properties at :root. Every colour
  in the file references a variable — no hex values scattered through the
  CSS. This is what makes rebranding it a two-minute job.
- Three type roles: a display face for headings, a readable sans for body,
  a mono for stamps, dates and figures.
- One accent colour used sparingly, for the things that are actually ours
  or actually urgent. If everything is highlighted, nothing is.
- Cards on a paper-toned background rather than boxes on white. Generous
  line height. Body text no smaller than 14px.
- Threat and status levels get a consistent colour language across every
  section — high/medium/low must look identical wherever they appear.
- Fully responsive. It will be read on a phone more often than not.

INTERACTIONS
------------
- Search boxes over the long tables, filtering as you type.
- Sortable columns on financial tables.
- Filter chips on the timeline and the category tables.
- Click a brand anywhere to open its detail panel plus auto-assembled story.
- A "jump to section" directory on the overview.
- Sticky nav so orientation never gets lost.

ASSISTANT
---------
Add a floating "Ask the Radar" button opening a chat panel that answers
strictly from the dashboard's own data. Serialise the arrays into a context
object at load. Offer role presets — General, Product, Marketing, Strategy,
Leadership — each with four seeded questions, so the first interaction needs
no thinking. Include a short profile of us in the context so answers are
framed from our position, and explicitly name our partners as allies so it
never files them as competitors.

EDITORIAL RULES
---------------
- Every figure carries its fiscal year and its source.
- Nothing invented. If it is not confirmable it is omitted or labelled.
- Where sources disagree, show the range and say they disagree.
- "Not found" is an acceptable and honest value.
- Flat, factual tone. No hype adjectives. The reader is smart and busy.
- Each entity gets a "latest move" line — the single most recent thing
  that happened to it. This is what makes a static file feel alive.
- Mark the file confidential and internal in the header and footer, and
  stamp it with a "Data as of" date in at least four places so a stale
  copy is obvious at a glance.

VERIFY BEFORE DELIVERING
------------------------
Extract the largest <script> block to a file and run `node --check` on it.
A syntax error renders the whole dashboard blank. Do not deliver unverified.
```

---

# PROMPT 3 — Ship it and keep it current

```
Package the dashboard for deployment and set up an automatic refresh.

1. Build a deploy folder: index.html plus a host config file carrying noindex
   and basic security headers (vercel.json), a robots.txt that blocks crawlers,
   and a README with setup steps. Initialise it as a git repo and commit.

2. Write REFRESH.md into the repo root — a runbook the scheduled task reads
   each run. It must specify: which data arrays carry news versus standing
   profiles, which companies and topics to research, the editorial rules,
   the syntax-check step, and the commit convention. Keeping this in the repo
   rather than in the schedule means changing what gets tracked is a file edit,
   not a task edit.

3. Create a scheduled task at the cadence in the brief. Each firing starts a
   fresh session with no memory, so the prompt must be fully standalone: clone
   the repo with a token, read REFRESH.md, research the window since the last
   "Data as of" date, edit the arrays, move the date stamps, syntax-check,
   commit and push. Vercel redeploys from the repo on its own.

4. Have each run report back with the two or three developments that actually
   matter, not a list of everything found.
```

**The credential caveat.** The scheduled session needs a token to push, and it
has to live in the task configuration — there is no separate secrets store.
Use a fine-grained token scoped to the single repo, contents read-write only,
with an expiry. If that trade is unappealing, the alternative is keeping the
canonical copy in cloud storage and having each run send you the file to deploy
manually — thirty seconds of work per cycle, no credential anywhere.

---

# Adapting to another industry

The seven organs hold everywhere. What changes is vocabulary and which
operating sections earn their place.

**Consumer / retail** — as built. Price–value map, channels, social signals and
community all pull weight because distribution and culture decide outcomes.

**B2B SaaS** — swap Channels for *Integrations & ecosystem*, Social Signals for
*Developer mindshare* (GitHub stars, docs quality, community size), and Community
for *Analyst positioning* (the Gartner/Forrester quadrant reality). Add a
*Pricing & packaging* section; in SaaS the packaging is the strategy. Watchlist
should track hiring signals — job postings leak roadmap earlier than anything else.

**Deep tech / hardware** — add *Patent landscape* and *Supply chain & capacity*.
Replace the price–value map with a *maturity versus unit-cost* chart, since the
question is never who is cheapest but who has crossed into manufacturable.
Money & Moves matters more than usual: in capital-intensive categories, funding
is the clearest signal of who survives.

**Services / agency** — replace Channels with *Account wins and losses*,
Playbook with *Positioning and pitch angles*, and add *Talent flows*, because in
services the people moving between firms is the actual competitive dynamic.

**Regulated markets — health, fintech, energy** — promote regulation from a
sub-theme to a full section with its own timeline. Add a *Compliance posture*
comparison. In these industries the regulatory calendar sets the roadmap for
everyone, and knowing who is ready for what is the entire game.

Whatever the industry, the two sections to build first are **Watchlist** and
**Us vs Field**. They are where the thinking lives. Everything else is
context that makes those two credible.

---

# What to expect, and what goes wrong

**It is roughly 1,500 lines and 150KB.** That is the right size — big enough to
be genuinely useful, small enough to regenerate. If it grows past about 2,500
lines, the scope rules are not being enforced.

**The research is 70% of the effort and 30% of the value.** The value is in the
comparison logic, the threat rankings, and the honest self-assessment. Do not
let a strong research pack become an excuse for a weak Watchlist.

**The self-assessment section will be too kind on the first pass.** Every one of
these is written by someone who wants their own position to look good. Ask
directly for the case against you, and treat the answer as the useful part.

**Consistent field shapes matter more than they seem.** The first version always
has one entity type with an extra field "just for this one." Six months later
that is the reason the refresh breaks. Keep the shapes uniform.

**Date stamps in four places is deliberate.** Header, overview footer, page
footer, and the JS context object. A radar that looks current but is not is
worse than no radar.

**Sources are load-bearing, not decoration.** Sixty-plus cited references sounds
excessive until the first time someone in a board meeting asks where a number
came from.
