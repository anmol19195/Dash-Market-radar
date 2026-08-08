DASH MARKET RADAR — GITHUB + VERCEL AUTO-DEPLOY
Snapshot: 7 Aug 2026 · Internal & confidential

WHAT'S IN HERE
  index.html       the full dashboard (self-contained, no build step)
  vercel.json      response headers — noindex + basic security
  robots.txt       blocks search-engine crawling
  REFRESH.md       runbook the every-2-days scheduled refresh follows
  BUILD-PROMPTS.md reusable kit for building a radar for another industry
  .git/            already initialised with commits — just add a remote


SETUP — DO THIS ONCE (about 5 minutes)

  1. CREATE AN EMPTY PRIVATE REPO ON GITHUB
     Name it e.g. "dash-market-radar". Do NOT add a README, .gitignore or
     licence — this folder already has commit history and GitHub will refuse
     the push if the repo has its own.

  2. PUSH THIS FOLDER TO IT
       cd dash-radar-deploy
       git remote add origin https://github.com/<you>/dash-market-radar.git
       git push -u origin main

     If GitHub asks for a password, it wants a token, not your account
     password. The token from step 4 works, or use the GitHub CLI / SSH.

  3. IMPORT THE REPO INTO VERCEL
       vercel.com > Add New… > Project > Import Git Repository > pick the repo
       Framework Preset:  Other
       Build Command:     leave empty
       Output Directory:  leave empty (defaults to the repo root)
       Install Command:   leave empty
       Then click Deploy.

     There is no build step — Vercel just serves index.html. First deploy
     takes well under a minute and gives you a URL ending in .vercel.app.
     From then on, every push to main redeploys automatically.

  4. CREATE A TOKEN FOR THE SCHEDULED REFRESH
       GitHub > Settings > Developer settings > Personal access tokens >
       Fine-grained tokens > Generate new token
       Repository access:  Only select repositories > dash-market-radar
       Permissions:        Repository permissions > Contents > Read and write
       Expiration:         90 days (set yourself a reminder to rotate)

     Nothing else needs this token, and you can revoke it any time from that
     same page. Send it and your repo URL back to Claude to finish wiring up
     the schedule.


HOW THE REFRESH WORKS ONCE WIRED UP

  Every 2 days at 08:00 IST a fresh Cowork session clones this repo, reads
  REFRESH.md, researches what changed in footwear since the last snapshot,
  edits index.html, syntax-checks it, then commits and pushes. Vercel
  redeploys itself. You get a short summary of what actually changed.

  To change what gets tracked, edit REFRESH.md and push. The schedule reads
  it fresh every run, so the task itself never needs touching.


A NOTE ON ACCESS

  This document is confidential. robots.txt and the noindex header keep it
  out of search results, but anyone with the URL can still read it.

  Vercel's access controls, as of August 2026:
    - Vercel Authentication is on every plan including Hobby, but on Hobby it
      only covers preview and generated deployment URLs — your production
      URL stays public. Protecting the production domain needs Pro or above.
    - Password Protection is Enterprise, or a $150/month add-on on Pro.

  So on a free plan the practical answer is: keep the .vercel.app URL private,
  don't link to it anywhere, and rely on the noindex headers. If the radar
  ever needs to be genuinely locked down, Pro with All Deployments protection
  is the cheapest real option.


NOTES
  - Charts load Chart.js from jsDelivr and fonts from Google Fonts, so the
    page needs internet access to render fully. Everything else is inline.
  - No environment variables, no dependencies, nothing to install.
