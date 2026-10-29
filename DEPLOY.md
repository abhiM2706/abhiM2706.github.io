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
Live at `abhim2706.github.io` within a minute.

### Option C — Netlify drop

No git at all: go to <https://app.netlify.com/drop> and drag the
`abhinav-site` folder onto the page. Instantly live at a random subdomain.
Fine for a quick look, worse for updating later.

## Chosen: `malv.dev` -> `abhinav.malv.dev`

Register `malv.dev` (Cloudflare Registrar or Porkbun both carry `.dev`; it is
HTTPS-only by design, which GitHub Pages satisfies). Then add one record at the
registrar's DNS panel:

```
Type:   CNAME
Name:   abhinav
Target: abhim2706.github.io
TTL:    auto
```

If the registrar is Cloudflare, leave the record on **DNS only** (grey cloud)
until GitHub has issued the certificate. Then run, from this folder:

```bash
gh api -X PUT repos/abhiM2706/abhiM2706.github.io/pages -f cname=abhinav.malv.dev
```

GitHub writes a `CNAME` file into the repo, verifies the record, and issues a
Let's Encrypt certificate within about an hour. Once `https://abhinav.malv.dev`
loads, enforce HTTPS:

```bash
gh api -X PUT repos/abhiM2706/abhiM2706.github.io/pages -F https_enforced=true
```

The old `abhim2706.github.io/` address keeps working and
redirects to the new domain.

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

- **The wardrobe.** One list per slot in the script: `TOPS` (the Depop tops,
  drawn), `FITS` x `WAYS` (drawn garments in 20 colourways, used for 1 roll in
  4), `PANTS` (shorts included, with a `kind`), `SHOES`, `CAPS`, `BEANIES`
  and `GLASSES`. `rollFit()` takes one of each - shirt, pants, shoes, at most
  one thing on the head (cap, beanie, headphones or a raised hood), glasses
  two times in five - and `dress()` puts it on the bust, the arm and the
  lookbook. Warm tops lean toward pants, tees toward shorts. `CAPS` is the one
  list not from the purchase history: swap the placeholders for the real
  ones. New purchase: add a row to the right list, with a `col` so the
  figure can wear it.

  Hovering the head opens the **lookbook**: the full-length figure in the same
  fit (the head and bust are the live ones, borrowed with `<use>`, so it blinks
  and looks around too) and the itemised list, over the spot where he stands.
  It is hover-only, so it is hidden under 860px; on a phone, tapping the head
  shows it for a moment. On a wide screen (1180px and up) he stands in the
  middle of the left column.

  All garments are drawn over `SH`, the shared shoulder path, which has a
  deliberately flat top around `y=102` spanning `x` 40 to 80 - that flat is
  what gives collars and plackets somewhere to sit. If you add a garment, keep
  its details inside that silhouette: it is not clipped, so anything wider just
  hangs off the shoulders. Useful x-spans: `y=106` is 34-86, `y=112` is 28-92,
  `y=118` is 26-94. The brows are hard-clamped to `y >= 40`, so headwear may
  come down to `y=39` and never collides with them.

- **Dialogue.** Each mood in `MOODS` has a `quips` pool and one line is drawn
  at random (never the same one twice in a row). Anything with `data-mood`
  can also carry its own lines: `data-say="one|two|three"`. The jobs,
  project cards, hobbies and reels all do. The **arm** points at whichever
  `.card`, `.job`, `.hobby` or `.reel` the cursor is on (or the reel on
  screen); its sleeve takes the shirt colour.

- **Off the clock** is its own view (`#hobbies`, the middle button up top),
  five `.hobby` cards with an icon and copy drawn from the college essays.
  Hover a card and he does that hobby where he stands, in the left column,
  and keeps doing it until you hover another card, the head, or press
  escape. The gym card plays whichever lift tab is selected; hovering a tab
  previews that lift.

- **Doing the hobbies.** `ACTS` in the script holds one entry per action
  (`violin`, `dj`, `code`, `serve`, `squat`, `bench`, `dead`), each with a
  `hint` for the keys (shown under the figure), a `mood`, a `setup(st)` that
  lays out the scene, a `tick(sec, st)` called every frame with seconds since
  the start, a `key(e, st)` the keyboard reaches while it plays, and an
  `auto(sec, st)` that presses the keys itself whenever yours have been quiet
  for a couple of seconds (and always on a touch screen). Space is a bow
  stroke / a rep, arrows scratch and space drops the beat, typing writes the
  code and enter runs it (24+ characters pass, fewer get a TLE), and the
  serve puts up a meter: space on the green is an ace, anything else and the
  ball comes down on his head. The decks show what he might be playing:
  `TRACKS` is a pull from the liked songs on his Spotify (read off the
  library in October 2026); one is drawn at random every seven seconds or
  on a drop. The three lifts are drawn side-on at a
  three-quarter angle with the bar loaded like a real one (`barbell()`:
  45 lb bar, then per side 45s, a 25, 10s, 5s until it adds up). The bar
  runs away from you at about 45 degrees, so it is foreshortened on screen
  and the plates show most of their face, stacked close like coins on a
  slant; near plates big and in front, far plates small and behind.

- **The figure.** `figureSVG()` is one rounded body under the live head,
  with two-segment arms placed by `setArm(side, x, y, bend)`. Every top has
  a full-length version for it (`FULL` by template, `FULL_TOP` for the
  hand-drawn ones): the torso from the shoulders to the hem, clipped to the
  body, and a `sleeve` spec the arm drawing reads (colour, cuff, stripe,
  bands, a camo blob). Pants carry their marks, pockets, drawstrings and
  seams; shoes their N, tiger stripes, Air Max waves, laces. The shoulders
  are one rounded yoke in the shirt over the arm roots, under the bust's
  collar, so there is no seam; the pointing arm on the bust comes out from
  behind its shoulder the same way.

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
