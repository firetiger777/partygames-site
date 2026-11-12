# polargamesinc.com — the Polar Games Inc website

This folder **is** the website. It is a static, dependency-free tree of HTML that gets published to
GitHub Pages and served at **https://polargamesinc.com** (PRD §34.8).

There is no build step. Open any file directly in a browser to preview it.

## Pages

| Path | Live URL | What it is |
|------|----------|------------|
| `index.html` | `/` | Polar Games Inc company home: who we are, the games, contact. Doubles as the Play Console **developer website**. |
| `support.html` | `/support.html` | Support + FAQ for the **Party Games** hub app (the private staging app, PRD §34.1). |
| `privacy.html` | `/privacy.html` | Privacy policy for the **Party Games** hub app (no networking, still accurate for the hub). |
| `cantsay/index.html` | `/cantsay/` | **Can't Say That!** app page: support, contact, how to play, FAQ, screenshots. This is the app's **Support URL**. |
| `cantsay/privacy.html` | `/cantsay/privacy.html` | **Can't Say That!** privacy policy. This is the app's **Privacy Policy URL** (required by both stores). |
| `join/index.html` | `/join/` | **Room invite landing** (PRD §35.1). Shows the room code from `?g=<game>&c=<CODE>` and forwards to the right store per platform. Only ever renders when the app is *not* installed: when it is, the OS intercepts the link first. |
| `og.png` | `/og.png` | 1200x630 link-preview card for the home page. Source: `_og/og.html`. |
| `cantsay/img/*.png` | | Web-sized (540x960) marketing screenshots. |
| `CNAME` | | Tells GitHub Pages the custom domain. Do not delete. |
| `.well-known/assetlinks.json` | `/.well-known/assetlinks.json` | **Android App Links** verification (PRD §35.6). Lists each package's **Play app-signing** SHA-256 plus the upload key. |
| `.well-known/apple-app-site-association` | same | **iOS Universal Links** (PRD §35.6). Copied to the site root too, where older iOS looks. No file extension, by Apple's rule. |
| `.nojekyll` | | 🛑 **Load-bearing. Never delete.** GitHub Pages runs Jekyll by default and Jekyll silently drops every path starting with `.` or `_`, which would 404 `/.well-known/…` and break App Link verification with no error anywhere. |

Each page is self-contained: its CSS lives in a `<style>` block, there is no shared stylesheet and
no external requests. Keep it that way, it is what makes the site trivially maintainable and instant
to load.

**One exception:** `join/index.html` uses a small inline `<script>`. Reading a query string and
branching on platform cannot be done statically, and GitHub Pages has no server to do it for us. It
is still dependency-free and makes no external requests, which is the part of the rule that matters.

### Testing the invite page

It is pure client-side logic, so headless Chrome with a spoofed UA is enough:

```bash
CHROME="/c/Program Files/Google/Chrome/Application/chrome.exe"
U='file:///c:/PartyGames/site/join/index.html?g=cant-say-online&c=ABCD'
"$CHROME" --headless=new --disable-gpu --no-sandbox --virtual-time-budget=2500 \
  --user-agent="Mozilla/5.0 (Linux; Android 14; SM-S146VL) Mobile Safari/537.36" \
  --dump-dom "$U" | grep -oE 'id="code"[^>]*>[^<]*|id="cta" href="[^"]*"'
```

Check all four branches: Android (Play URL gains `&referrer=c%3DABCD`), iOS, desktop, and a missing
or malformed `c` (must fall back to "Ask your host for the code", never a trimmed partial).

## Where the assets come from

The screenshots are the **same art as the store listings**, rendered smaller for the web. They are
regenerated from the store screenshot HTML, not resized by hand:

```bash
# 540x960 web screenshots (half of the 1080x1920 Play phone profile)
cd store/cantsay/screenshots
CHROME="/c/Program Files/Google/Chrome/Application/chrome.exe"
for n in 01 02 03 04 05 06; do
  "$CHROME" --headless=new --disable-gpu --no-sandbox --hide-scrollbars \
    --force-device-scale-factor=0.5 --window-size=1080,1920 \
    --screenshot="$(cygpath -w "/c/PartyGames/site/cantsay/img/$n.png")" \
    "file:///$(cygpath -w "$PWD/_html/play-phone/$n.html")"
done

# 1200x630 link-preview card
cd /c/PartyGames/site/_og
"$CHROME" --headless=new --disable-gpu --no-sandbox --hide-scrollbars \
  --force-device-scale-factor=1 --window-size=1200,630 \
  --screenshot="$(cygpath -w /c/PartyGames/site/og.png)" \
  "file:///$(cygpath -w "$PWD/og.html")"
```

If the store screenshots are re-rendered (`store/cantsay/screenshots/generate.mjs` +
`render.sh`), re-run the first block so the website matches the listings.

## Publishing

The live host is the **`firetiger777/partygames-site`** GitHub repo, served by GitHub Pages. This
folder is the source of truth; that repo is a publish target.

```bash
bash scripts/deploy-site.sh "what changed"

# publish the content WITHOUT claiming polargamesinc.com (use until DNS is pointed)
SKIP_CNAME=1 bash scripts/deploy-site.sh "what changed"
```

The script clones the site repo, replaces its contents with this folder, commits, and pushes.
GitHub Pages redeploys within a minute or so.

**Current state (2026-08-09): live at https://polargamesinc.com.** DNS, the custom domain, the TLS
certificate (apex + `www`), and Enforce HTTPS are all done. The one-time setup below is recorded for
reference and for the next domain; you do not need to repeat it.

## One-time setup

> ⚠️ **Do the DNS step BEFORE the first deploy.** `CNAME` tells GitHub Pages to serve at
> polargamesinc.com *and to redirect* `firetiger777.github.io/partygames-site/…` there. Those
> github.io URLs are the ones currently on file in App Store Connect, so publishing the CNAME while
> the domain still has no DNS would redirect them to a host that does not resolve, i.e. it would
> break the live privacy/support pages. DNS first, then `deploy-site.sh`.

**1. Dynadot DNS** — ✅ **done 2026-08-09** via `node scripts/dynadot-dns.mjs set-github-pages`
(needs `. ./.env.store.local` first). Verify any time with `node scripts/dynadot-dns.mjs get`.

To do it by hand instead (Domains → polargamesinc.com → DNS Settings): delete any
parking/forwarding records first, then set:

| Type | Host / Name | Value |
|------|-------------|-------|
| A | `@` (blank / root) | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |
| AAAA | `@` | `2606:50c0:8000::153` |
| AAAA | `@` | `2606:50c0:8001::153` |
| AAAA | `@` | `2606:50c0:8002::153` |
| AAAA | `@` | `2606:50c0:8003::153` |
| CNAME | `www` | `firetiger777.github.io` |

The AAAA records are optional but recommended. Do **not** point `@` at a CNAME, apex domains
cannot use one.

**2. GitHub Pages** — ✅ **done 2026-08-09.** Publishing `CNAME` sets the custom domain by itself;
Enforce HTTPS was turned on via the API once the certificate was approved:

```bash
gh api repos/firetiger777/partygames-site/pages \
  --jq '{cname,status,https_enforced,cert:.https_certificate.state}'   # check
gh api -X PUT repos/firetiger777/partygames-site/pages -F https_enforced=true
```

GitHub clears `https_enforced` while a new certificate is pending, so set it *after* the cert state
reads `approved`.

**3. Verify:**

```bash
curl -sI https://polargamesinc.com/ | head -1
curl -sI https://polargamesinc.com/cantsay/privacy.html | head -1
```

GitHub also redirects the old project URLs (`firetiger777.github.io/partygames-site/...`) to the
custom domain, so the URLs already registered in App Store Connect keep resolving. Update them to
the `polargamesinc.com` equivalents anyway, see below.

## Store fields that point here

Update these once the domain resolves. The canonical values are already written into the store
kits (`store/*/apple/asc-upload.mjs`, `store/*/**/metadata.md`).

| Where | Field | Value |
|-------|-------|-------|
| Play Console → developer account | Website | `https://polargamesinc.com/` |
| Play Console → Can't Say That! → Store listing | Privacy Policy | `https://polargamesinc.com/cantsay/privacy.html` |
| App Store Connect → Can't Say That! | Support URL | `https://polargamesinc.com/cantsay/` |
| App Store Connect → Can't Say That! | Privacy Policy URL | `https://polargamesinc.com/cantsay/privacy.html` |
| App Store Connect → Party Games (hub) | Support / Privacy | `https://polargamesinc.com/support.html` · `https://polargamesinc.com/privacy.html` |
