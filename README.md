# Verity — Lossless Gemini Watermark Remover

A single-file, 100% client-side web app that removes the **Google Gemini corner
sparkle watermark** from 1K and 2K images *losslessly* — by exactly inverting the
alpha blend instead of smearing it away with AI inpainting.

**Live demo:** `https://username.github.io/verity-watermark-remover/`
*(replace with your own URL — see [Before you deploy](#before-you-deploy))*

---

## Why it's different

Most "watermark removers" crop the corner, inpaint with a diffusion model, or
clone-stamp over it. All three **destroy pixels**. Verity does the opposite: it
solves for the original pixel values.

Gemini composites its watermark as a known alpha blend:

```
I_wm = (1 − α) · I + α · 255
```

where `α` is a fixed, calibrated alpha map baked into the app for both the
**56×56 (48 px star, α_max = 0.306)** and **104×104 (96 px star, α_max = 0.502)**
stamps. Verity inverts that equation exactly:

```
I = (I_wm − α · 255) / (1 − α)
```

Because the blend is inverted algebraically rather than approximated, the
result is **bit-for-bit identical** everywhere outside the watermark, and sharp
ink lines passing *behind* the star come back razor-crisp instead of blurred.

> Removes the **visible ✦ overlay only**. Invisible SynthID and C2PA metadata
> are not modified.

## Features

- **Exact reverse-alpha unblending** calibrated for 1K and 2K Gemini exports
  across all aspect ratios (1:1, 16:9, 9:16, 3:4 and scaled variants)
- **~99.9% of pixels untouched** — bit-for-bit identical outside the stamp
- **Zero line blur** — no AI inpainting smudge, no clone-stamp artefacts
- **Bulk processing** — drag in dozens of images, export one ZIP
- **4× before/after loupe** — pixel-level proof in the corner of the stage
- **Manual calibration** — corner, stamp size, offsets, opacity gain, de-ringing
- **Lossless PNG export** at the original dimensions
- **100% private** — images never leave the device; no server, no upload, no
  telemetry
- **Mobile-first layout** — the dock and batch bar are content-sized and
  centred (`width: fit-content` + `margin: 0 auto`), so they hug their
  content and can never overflow the viewport. Below 860px the dock drops
  to icons plus short labels (`PNG`, `ZIP (5)`) and the batch bar compresses
  into a scrolling filmstrip. Verified from 320px to 430px wide, plus
  landscape phones.

## Supported formats

| Format | Notes                                    |
| ------ | ---------------------------------------- |
| PNG    | Native, preferred                        |
| JPEG   | De-ringing filter reduces block artefacts |
| WebP   | Supported                                |

---

## Before you deploy

The repo ships with the placeholder URL `username.github.io/verity-watermark-remover`.
**Replace it** — otherwise canonical tags, Open Graph, the sitemap and the
robots.txt all point at a URL that isn't yours (and won't match your live site).

### Option A — one find-and-replace (recommended)

Search the repo for `username.github.io/verity-watermark-remover` and replace it
with your real URL. It appears in:

| File                   | What it's for                                  |
| ---------------------- | ---------------------------------------------- |
| `index.html`           | `<link rel="canonical">`, `og:url`, `og:image`, JSON-LD `@id`s |
| `sitemap.xml`          | The URL submitted to search engines             |
| `robots.txt`           | `Sitemap:` directive                           |
| `README.md`            | The demo link                                  |

Also replace `github.com/username/verity-watermark-remover` in the
`license` field of the JSON-LD in `index.html`.

### Option B — custom domain

If you're using a custom domain (e.g. `verity.example.com`):

1. Create a `CNAME` file in the repo root containing **only** your domain
   (no `https://`, no trailing slash).
2. Add a `CNAME` DNS record for your host pointing to
   `username.github.io`.
3. Enable **Enforce HTTPS** in *Settings → Pages*.

---

## Deploy to GitHub Pages

### Option 1 — GitHub Actions (recommended)

1. Create a new repository named `verity-watermark-remover` and push this folder
   to the `main` branch.
2. Go to **Settings → Pages → Build and deployment → Source** and choose
   **GitHub Actions**.
3. Push a commit (or run the *Deploy to GitHub Pages* workflow manually from
   the **Actions** tab).

The included workflow (`.github/workflows/pages.yml`) publishes the repo root
as-is, with `.nojekyll` preventing Jekyll from mangling anything.

### Option 2 — deploy from a branch

1. Push to `main`.
2. **Settings → Pages → Source** → **Deploy from a branch** → `main` /
   `/ (root)`.
3. Save. GitHub serves the files as static content within ~1 minute.

### Option 3 — a custom domain

Use the [custom domain steps](#option-b--custom-domain) above; the rest is
identical.

---

## Repository layout

```
verity-watermark-remover/
├── index.html              # the entire app — HTML + CSS + JS in one file
├── 404.html                # styled not-found page (noindex)
├── robots.txt              # crawl directives + sitemap pointer
├── sitemap.xml             # for search engines
├── CNAME                   # optional — only if you use a custom domain
├── .nojekyll               # serve the repo as-is, no Jekyll processing
├── LICENSE                 # MIT
├── assets/
│   ├── og-image.jpg        # 1200×630 social share card
│   ├── favicon.svg         # vector mark
│   ├── favicon-32.png      # raster favicon
│   ├── favicon-mono.svg    # Safari pinned-tab icon
│   ├── apple-touch-icon.png
│   └── site.webmanifest    # PWA metadata
└── .github/workflows/
    └── pages.yml           # GitHub Pages deployment
```

## Local development

It's a static page — any server works:

```bash
python -m http.server 8000
# then open http://localhost:8000
```

Opening `index.html` directly with `file://` also works, but a server is
required for the PWA manifest and some browsers' clipboard APIs.

---

## SEO notes

Everything technical is already in place:

- `<link rel="canonical">`, `og:url`, `og:image` (+ `og:image:width/height/type/alt`)
- `twitter:card = summary_large_image` with an absolute image URL
- `robots` / `googlebot` directives with `max-image-preview:large`
- **JSON-LD `@graph`** with `Organization`, `WebSite`, `WebPage` and
  `WebApplication` (`applicationCategory`, `offers`, `featureList`,
  `softwareVersion`, `isAccessibleForFree`, `screenshot`)
- `robots.txt` + `sitemap.xml` wired together
- `site.webmanifest` with maskable icons for PWA installability
- A `404.html` that is `noindex` so it never competes with the main page

**One thing to add once the site is live:** verify ownership in Google Search
Console and Bing Webmaster Tools, then submit `sitemap.xml`. If you want a
search snippet preview, paste the verification token into a `<meta
name="google-site-verification" content="...">` tag in `index.html`.

## License

MIT — see [LICENSE](LICENSE).

Verity is an independent project and is not affiliated with, endorsed by, or
supported by Google.
