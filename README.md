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
| `og.png` | `/og.png` | 1200x630 link-preview card for the home page. Source: `_og/og.html`. |
| `cantsay/img/*.png` | | Web-sized (540x960) marketing screenshots. |
| `CNAME` | | Tells GitHub Pages the custom domain. Do not delete. |

Each page is self-contained: its CSS lives in a `<style>` block, there is no shared stylesheet, no
JavaScript, and no external requests. Keep it that way, it is what makes the site trivially
maintainable and instant to load.

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
```

The script clones the site repo, replaces its contents with this folder, commits, and pushes.
GitHub Pages redeploys within a minute or so.

## One-time setup

> ⚠️ **Do the DNS step BEFORE the first deploy.** `CNAME` tells GitHub Pages to serve at
> polargamesinc.com *and to redirect* `firetiger777.github.io/partygames-site/…` there. Those
> github.io URLs are the ones currently on file in App Store Connect, so publishing the CNAME while
> the domain still has no DNS would redirect them to a host that does not resolve, i.e. it would
> break the live privacy/support pages. DNS first, then `deploy-site.sh`.

**1. Dynadot DNS** (Domains → polargamesinc.com → DNS Settings). Delete any parking/forwarding
records first, then set:

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

**2. GitHub Pages** (github.com/firetiger777/partygames-site → Settings → Pages):
set **Custom domain** to `polargamesinc.com`, save, wait for the DNS check and the TLS certificate
to be issued (usually minutes, can take up to a few hours), then tick **Enforce HTTPS**.

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
