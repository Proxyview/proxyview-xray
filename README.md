# Proxyview Lens

Retrospective portfolio assurance, client-facing.

## What to put on GitHub

**This whole repository.** Not a single HTML file.

`index.html` is the page source. It is an artifact fragment — it begins at
`<title>` and carries no `<!doctype>`, `<html>`, `<head>` or `<body>`, because
the Claude artifact platform supplies those. A browser will render it anyway,
which is why nothing looks obviously wrong, but it arrives with no character
encoding, no viewport and no favicon.

**Never deploy `index.html` directly.** Netlify runs the build below and
publishes `dist/`, which `netlify.toml` already configures. Push to GitHub and
the deploy takes care of itself.

    proxyview-lens/
      index.html                      page source
      netlify.toml                    build command, publish dir, headers
      package.json
      tools/build-site.mjs            wraps the fragment into a real document
      netlify/functions/quota.mjs     the free allowance, held server-side
      tests/                          lint, smoke, Selenium, Playwright
      dist/                           built output, git-ignored

## Build

    npm run build        # writes dist/index.html and dist/favicon.svg

No dependencies. It uses only Node builtins, so Netlify needs nothing
installed.

## Test

    npm install          # only the test tools; the site itself has no deps
    npx playwright install chromium
    npm test

Four suites — lint, structure, and the client journey in two browsers. They
load the built document, so what is tested is what gets served. The Selenium
suite skips itself when `tests/bin/chromedriver` is absent, which it is on a
fresh clone; Playwright covers the same journey.

`npm install` is needed for the tests and for nothing else. Netlify never runs
it: the build uses Node builtins only, so a deploy installs nothing.

## The free allowance

Five cases per **organisation**, taken from the work-email domain rather than
the person, so a second colleague does not get another five.

A page cannot enforce this on its own. Whatever it stores, the reader can
clear, and an incognito window starts clean. The count that decides anything
lives in `netlify/functions/quota.mjs`, on Netlify Blobs, which needs no
provisioning. The page keeps a mirror so the meter renders instantly; when the
server answers, the server wins.

When the function cannot be reached the client is not locked out — that would
punish them for our outage — and the page says the count is unconfirmed.

## Where submissions go

Netlify Forms, which is **off by default** for sites created since April 2023.
Enable it at Forms → Enable form detection, then deploy again, then add an
email notification at Forms → Submission notifications. Three forms should
appear: `lens-signup`, `lens-case` and `lens-contact`.

To bypass Netlify Forms entirely, set `ENDPOINT` near the top of the script in
`index.html` to a hosted form endpoint. Nothing else changes.

## The address

`www.lens.getproxyview.com`, set in one place — `SITE.url` in
`tools/build-site.mjs`. Change it there and the canonical tag and the link
preview both follow.

**As at 24 September, `lens.getproxyview.com` resolves and
`www.lens.getproxyview.com` does not.** Add the second record so the address in
`SITE.url` works:

    Type: CNAME   Host: lens       Value: <the name Netlify shows>
    Type: CNAME   Host: www.lens   Value: <the name Netlify shows>

Then attach both in Netlify under Domain management and set one as the primary
domain. Netlify redirects every other attached alias to the primary, so a
shared link, a search result and an analytics row agree on a single address.

Canonicalisation is deliberately not done with a redirect rule in
`netlify.toml`. A rule there fires whether or not its destination exists, so a
bare-to-www redirect written before the `www.lens` record was added would send
the only working address to one that does not resolve and take the site down.
Netlify's primary-domain setting can only point at a domain attached to the
site, so it cannot do that.

Worth knowing: `www.` on a subdomain is redundant, and `lens.getproxyview.com`
is the conventional form. To drop it, change `SITE.url` — one line.

## Devices

Three treatments, not one breakpoint.

| Width | Treatment |
|---|---|
| up to 834 px | phone and iPad portrait: the rail collapses behind a control carrying the case count |
| 835 to 1120 px | tablet: two columns, narrower rail |
| 1121 px and up | desktop: as drawn |

Tested at nine sizes from iPhone SE to a 1680 px desktop: the gate, the report,
the upload screen, the paywall and the contact form each checked for
horizontal overflow, touch-target height and the rail behaving as that width
calls for. Inputs are 16 px on a phone, which is what stops iOS zooming the
whole page when a field takes focus.

## Telling which build is live

Every build stamps a date and time into the top-right of the page. Open the
console and run `LENS.diag()` for what was posted, where, and what came back.
