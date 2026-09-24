ش# dorker.sh — Search-Engine Dork Console

A single-file, client-side HTML tool that generates **Google/Bing dork queries** for a target domain. You type a domain, it builds a list of ready-to-click search URLs (`site:`, `ext:`/`filetype:`, `inurl:`, `intitle:`) and opens whichever one you pick in a new tab. Nothing is sent anywhere except the search engine itself — there's no backend, no tracking, no data collection.

> ⚠️ **Use responsibly.** This tool only *builds search-engine URLs* — it doesn't scan, exploit, or access anything on its own. Still, running these queries against a domain you don't own or don't have explicit permission to test (e.g. no bug-bounty scope, no pentest authorization) can violate that organization's terms of service or local law. Always confirm you're allowed to recon a target before pointing this at it.

---

## What it does

Given a domain (e.g. `example.com`), the console builds 11 pre-written dork queries and lets you launch any of them in Google or Bing with one click.

| # | Query pattern | What it's trying to find |
|---|---|---|
| 1 | `site:domain inurl:&` | Any indexed page whose URL contains a query string (`?a=1&b=2`) — a quick way to see which pages take parameters at all. |
| 2 | `site:domain ext:aspx \| ext:asp \| ext:jsp \| ext:html \| ext:htm` | Legacy or server-rendered pages, useful for fingerprinting the tech stack (classic ASP/JSP often signals older, less-maintained infrastructure). |
| 3 | `site:domain ext:log \| ext:txt \| ext:conf \| ext:env \| ext:bak \| ext:git \| ...` | Accidentally indexed **config files, backups, logs, and version-control artifacts** — one of the most common sources of leaked secrets. |
| 4 | `site:domain inurl:url= \| inurl:return= \| inurl:next= \| inurl:redir= inurl:http` | Parameters that look like **open-redirect** candidates. |
| 5 | `site:domain inurl:http \| inurl:url= \| inurl:path= \| ... inurl:&` | Broader sweep for any parameter that might carry a URL or file path (SSRF / LFI candidates). |
| 6 | `site:domain inurl:config \| inurl:env \| inurl:backup \| inurl:admin \| inurl:php` | Pages/paths that *sound* administrative or configuration-related. |
| 7 | `site:domain inurl:email= \| inurl:phone= \| inurl:password= \| inurl:secret= inurl:&` | Parameters that may leak **PII or credentials** directly in the URL. |
| 8 | `site:domain inurl:apidocs \| inurl:swagger \| inurl:api-explorer` | Exposed **API documentation** (Swagger/OpenAPI UIs are frequently left public and reveal the entire API surface). |
| 9 | `site:domain inurl:cmd \| inurl:exec= \| inurl:query= \| inurl:run= \| ... inurl:&` | Parameters that resemble **command/query execution** endpoints — potential RCE/SQLi surface. |
| 10 | `site:domain inurl:(unsubscribe\|register\|feedback\|signup\|...)` | Generic user-facing functionality pages, useful for mapping the app's feature set. |
| 11 | `site:domain intitle:"Welcome to Nginx"` | Default/unconfigured **Nginx welcome pages** — a sign of a forgotten or misconfigured host. |

None of these queries do anything by themselves beyond asking Google/Bing "what have you indexed that matches this pattern?" — they only surface what the search engine has already crawled and cached publicly.

---

## Google vs. Bing: why the two modes differ

The tool writes every dork in **Google syntax first**, then auto-translates it for Bing when you switch tabs. The two engines are *not* interchangeable:

| | Google | Bing |
|---|---|---|
| File-type operator | `ext:` | Bing doesn't support `ext:` — it must be `filetype:`. The tool rewrites this automatically. |
| OR logic | A bare pipe `\|` works and stays scoped inside the query | Bing does **not** reliably scope `site:` across an un-parenthesized `\|` chain — everything after the first `\|` can be treated as a *separate, unscoped* query, silently leaking results from unrelated domains. The tool fixes this by pulling `site:` out front and rewriting the rest as `site:domain (a OR b OR c)`. |
| `site:` value | Must be a bare domain (`example.com`), never a full URL with `https://` | Same rule applies |

If you ever hand-edit a query outside this tool, keep these two differences in mind — copy-pasting a Google-style dork straight into Bing (or vice versa) is the single most common reason a dork "returns nothing" when results clearly exist.

---

## Common problems you'll run into

- **Empty results ≠ nothing exists.** A dork returning zero hits only means the search engine hasn't indexed a matching page — not that the underlying issue doesn't exist. Search engines skip pages blocked by `robots.txt`, pages requiring auth, and anything behind `noindex`.
- **Stale index / cache lag.** A file might have been indexed months ago and removed since. Always verify a hit is still live before treating it as a finding.
- **CAPTCHAs and rate-limiting.** Firing off many dorks quickly (especially with automated tooling) will trigger Google's "unusual traffic" CAPTCHA wall or Bing's throttling. This tool intentionally opens one query per click, in a new tab, to keep usage manual and low-volume.
- **Operator limits per query.** Google silently drops or ignores extra operators once a query gets too long or too complex — if a dork with many `\|` chains looks like it's not filtering, try splitting it into smaller queries.
- **`site:` scoping bugs (Bing specifically).** As noted above, an unparenthesized `\|` chain on Bing can leak unrelated domains into results — always double check the `site:` operator is actually still applied to every clause.
- **False positives from generic keywords.** Broad dorks (e.g. `inurl:admin`, `inurl:backup`) match plenty of harmless pages (marketing copy containing the word "admin", blog posts about backups, etc.) — treat hits as *leads to manually verify*, not confirmed issues.
- **URL encoding quirks.** Special characters in the domain or in hand-added operators (spaces, quotes, `&`) need proper URL-encoding or the query breaks; this tool handles encoding for you via `encodeURIComponent`, but be careful if you copy a raw query elsewhere.
- **Engine-specific operator support drifts over time.** Both Google and Bing periodically deprecate or restrict operators (e.g. Google has scaled back `inurl:`/`intext:` reliability at various points). If a dork that used to work stops returning anything, check whether the operator itself is still supported before assuming the target has no exposure.

---

## Usage

1. Open `index.html` in any browser — no build step, no dependencies beyond a Google Fonts import.
2. Enter a target domain in the input field.
3. Toggle **Google** / **Bing** to switch query syntax.
4. Click any line to open that dork in a new tab.

---

## Credits

Dork patterns adapted from public dorking references, including [taksec/google-dorks-bug-bounty](https://taksec.github.io/google-dorks-bug-bounty/).
