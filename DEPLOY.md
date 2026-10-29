# Getting this online at your own domain

`index.html` is the whole site. One file, no build step, no dependencies —
the only thing it fetches is the Google Fonts stylesheet. You can open it
by double-clicking it, and anything that serves a static file can host it.

---

## First: `abhinav.malv` isn't a domain you can buy

`.malv` is not a top-level domain. There are ~1,200 TLDs and `.malv` isn't
one of them, so no registrar can sell you `abhinav.malv` — it simply won't
resolve anywhere.

Creating a brand-new TLD means applying to ICANN for it. The current round's
application window opened 30 April 2026 and **closed 12 August 2026**, and the
base application fee is **USD 227,000** before legal and operating costs. So
that path is closed and wasn't realistic anyway.

### The closest thing you can actually register

`.in` is India's ccTLD, open to anyone, about $10–12/year:

| Buy this | Your site lives at | Reads as |
|---|---|---|
| **`malv.in`** | `abhinav.malv.in` | "abhinav dot malv dot in" — one character off your original |
| **`malv.in`** | `malv.in` | short, clean, memorable |

With `malv.in` you get *both*: the apex for the site and `abhinav.` as a
subdomain if you want the full `abhinav.malv.in` reading.

### Other options worth checking

- `abhinavmalviya.com` — the boring, correct, never-regret-it choice
- `malviya.dev` / `abhinav.dev` — `.dev` is Google-run, HTTPS enforced, ~$12/yr
- `malv.sh`, `malv.io`, `malv.me` — short, more expensive ($30–60/yr)

Check availability at <https://instantdomainsearch.com> — it checks every TLD
as you type.

---

## Step 1 — Buy the domain

Use **Cloudflare Registrar** (<https://dash.cloudflare.com>). It sells at
wholesale cost with no markup and no first-year-cheap/renewal-expensive trick,
and includes WHOIS privacy free. You need a Cloudflare account first; you can
also register elsewhere and transfer in later.

Alternatives: **Porkbun** (cheap, good UI, supports more TLDs) or **Namecheap**.
Avoid GoDaddy — low intro price, high renewals, constant upsells.

> Note: `.in` requires a registrar that supports it. Cloudflare does; confirm
> at checkout.

## Step 2 — Put the site on a host

Pick one. All three are free for a site this size.

### Option A — Cloudflare Pages (recommended if you bought at Cloudflare)

DNS and hosting in one place, so step 3 is nearly automatic.

```bash
cd ~/Desktop/abhinav-site
git init -b main
git add -A
git commit -m "Personal site"
gh repo create abhinav-site --public --source=. --push
```

Then in the Cloudflare dashboard: **Workers & Pages → Create → Pages →
Connect to Git**, pick `abhinav-site`, and leave the build settings empty
(no build command, output directory `/`). It deploys on every push to `main`.

### Option B — GitHub Pages

Simplest if you want everything on GitHub. Same `git` commands as above, then
in the repo: **Settings → Pages → Source: Deploy from a branch → `main` / root**.
Live at `abhim2706.github.io/abhinav-site` within a minute.

### Option C — Netlify drop

No git at all: go to <https://app.netlify.com/drop> and drag the
`abhinav-site` folder onto the page. Instantly live at a random subdomain.
Fine for a quick look, worse for updating later.

## Step 3 — Point the domain at the host

### If you used Cloudflare Pages

In your Pages project: **Custom domains → Set up a custom domain** → type
`malv.in` (and again for `abhinav.malv.in` if you want both). Cloudflare
writes the DNS records itself. Done.

### If you used GitHub Pages

In your domain's DNS settings, add:

**For an apex domain** (`malv.in`) — four `A` records, all with name `@`:

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

**For a subdomain** (`abhinav.malv.in`) — one `CNAME`:

```
Name:   abhinav
Target: abhim2706.github.io
```

If the records sit behind Cloudflare's proxy (orange cloud), switch them to
**DNS only** (grey cloud) until GitHub has issued the certificate, then turn
the proxy back on if you want it.

Then back in **Settings → Pages**, enter the domain in **Custom domain** and
save. GitHub writes a `CNAME` file into the repo — leave it there.

### If you used Netlify

**Site settings → Domain management → Add custom domain**, then follow its
instructions (it will give you either a `CNAME` or nameservers to use).

## Step 4 — Wait, then turn on HTTPS

DNS propagates in minutes to a few hours. Check it with:

```bash
dig +short malv.in
```

Once that returns the right addresses, tick **Enforce HTTPS** (GitHub Pages)
or confirm the certificate is active (Cloudflare/Netlify). All three issue a
free Let's Encrypt certificate automatically — you never buy one.

---

## Updating the site later

Edit `index.html`, then:

```bash
cd ~/Desktop/abhinav-site && git add -A && git commit -m "Update" && git push
```

Cloudflare Pages and GitHub Pages both redeploy on push.

---

## Things you might want to change in `index.html`

- **The face.** The whole head is one inline `<svg>` near the top of the
  `<body>`, and every expression is a row in the `MOODS` object in the script.
  Each row is just target numbers - `lid` (eye openness), `browY`/`browTilt`,
  `curve` (smile to frown), `open` (mouth), `blush`, `tilt`, and the `quip`
  string that shows in the little bubble. Change a number, reload, see it.

  One constraint worth knowing: `browY` can't go much past `-6`. The brows sit
  on a strip of forehead between the hairline (y~35) and the eyes (y~61), and
  a bigger raise walks them up into the hair, where they disappear. If you want
  more range, raise the hairline in the hair `<path>` first.

- **The wardrobe.** `FITS` holds 19 garments and `WAYS` 20 colourways, so there
  are 380 base combinations before accessories: glasses (wire, black or
  tortoise), a gold hoop, a chain, a beanie, headphones or a scarf. One fit is
  rolled on every page load, and clicking the head pulls another. Each garment
  is a function returning an SVG string built from three colors: `main`, `alt`
  (shading and collars) and `trim` (contrast). Accessories pick their own
  colourway so they contrast with the garment. "hood up" is the one garment
  with an `over` part drawn in front of the face; it also suppresses headwear.

  All garments are drawn over `SH`, the shared shoulder path, which has a
  deliberately flat top around `y=102` spanning `x` 40 to 80 - that flat is
  what gives collars and plackets somewhere to sit. If you add a garment, keep
  its details inside that silhouette: it is not clipped, so anything wider just
  hangs off the shoulders. Useful x-spans: `y=106` is 34-86, `y=112` is 28-92,
  `y=118` is 26-94. The brows are hard-clamped to `y >= 40`, so headwear may
  come down to `y=39` and never collides with them.

- **The closet.** `TOPS` is every top from your Depop purchase history (54,
  out of 90 non-refunded orders), each drawn on one of the shared templates in
  its own colours with a chest mark where the piece has one; seven are drawn
  by hand to match the listing photo. Colours were checked against the
  listing photos; the one entry still marked `guess: true` (Grand Canyon long
  sleeve) is the only guess left.
  `SHORTS`, `PANTS`, `SHOES` and `SHADES` are the rest of the history; the bust
  can't show them, so `closetOutfit()` names them in the label (warm tops lean
  toward pants, tees toward shorts). Three quarters of rolls come from the
  closet; set that `0.75` in `rollFit()` to `1` for closet-only. New purchase:
  add a row to the right list.

- **The reels.** The `Reels` button (or opening the page at `#reels`) swaps the
  profile for a scroll-snap deck, one project per screen, in `<section
  id="reels">`. Each `<article class="reel">` has a `data-mood` the head takes
  on when that reel is on screen, an animated inline SVG in `.stage`, and the
  copy under it. Only the reel on screen animates: everything inside `.art` is
  `animation-play-state: paused` until the IntersectionObserver marks the reel
  `.live`, so a phone is never running eight clips at once. Arrow keys, j/k and
  the pips on the right step between reels. To add one, copy an article, give
  it a mood, and keep SVG captions inside the 200-unit viewBox (`.lbl` is 8
  units a character at most, `.lbl.big` about 6.3 - roughly 35 and 26 chars).
  `prefers-reduced-motion` shows every clip in its finished state instead.

- **Skin and hair colors** are the `--skin`, `--skin-dk`, and `--hair` tokens
  in the `:root` block, with separate dark-mode values further down.
- **A project reacts to hover** through its `data-mood="..."` attribute. Point
  it at any key in `MOODS`.
- **Your phone number is deliberately not on the page.** It's on your resume,
  but a public page gets scraped. Add it if you want it — I left it off by
  default.
- **Resume PDF.** If you want a download link, drop `AbhiResume.pdf` in this
  folder and add a chip in the "Say hello" section:
  `<a class="chip" data-mood="proud" href="AbhiResume.pdf">Resume</a>`
