# CLAUDE.md — eriksheridan.com

## What this is

The **apex domain**, which serves a one-page *menu* pointing at five other addresses.
It is not the resume any more — the resume moved to its own subdomain in September 2026.

Erik is not an engineer. Handle tooling, DNS and setup for him rather than handing over
commands, and explain changes by their consequence ("visitors will see X") rather than by
mechanism.

## The six addresses

| Address | Repo | Host | What it is |
|---|---|---|---|
| `eriksheridan.com` | `erikrocks/eriksheridan.com` (this one) | GitHub Pages | The menu / hub |
| `resume.eriksheridan.com` | `erikrocks/resume.eriksheridan.com` | GitHub Pages | The resume |
| `kudr.eriksheridan.com` | `erikrocks/kitty-unicorn-game` | GitHub Pages | Vivian's browser game |
| `ebaapl.eriksheridan.com` | `erikrocks/ebaapl` (private) | **Vercel** | NCAA pick'em app |
| `bubble.eriksheridan.com` | `erikrocks/bubble` | GitHub Pages | Pixel-art bubble game |
| `gametimetalk.com` | `erikrocks/ebaapl` (private) | **Vercel** | Spoiler-free football chat (beta) |

All four GitHub Pages sites: branch `main`, path `/`, custom domain set via a `CNAME`
file in the repo root, certificate approved, **Enforce HTTPS on**. Push to `main` and it
redeploys; nothing else to configure.

EBAAPL and GameTimeTalk are the odd ones out — Next.js on Vercel, auto-deploy on push,
DNS pointing at Vercel rather than GitHub. Don't try to manage either through the Pages API.

**GameTimeTalk shares EBAAPL's repo and Vercel project.** One deployment serves both sites
and the *hostname* decides which one you get; there is no `gametimetalk` repo (splitting it
out is on the backlog). It is also the only entry on the hub that is not an
`eriksheridan.com` subdomain — it has its own domain, registered at Squarespace like the
others. Link the site, never a repo.

Don't describe GameTimeTalk as finished or polished. Two things are deliberately not said
in public: which sports-data feed it uses, and any detail of its internals or known gaps.
Fine to say: free, no accounts, works on phones.

The GameTimeTalk session asked for an explicit "Still a beta" in the hub entry. It was
there until 7 Nov 2026, when **Erik removed it** — his reasoning: everything on this page
is in progress, and the "What I'm building" header already says so. Don't put it back on
the strength of the original request; it's his site and he has ruled on it. The
non-disclosure rules above are unaffected and still stand.

## Why subdomains and not `eriksheridan.com/kudr`

The apex is served by a **project** repo — there is no `erikrocks.github.io` user-site repo.
A custom domain on a project repo serves ONLY that repo, so a path pointing at a different
repo always 404s. This was tried and it cannot work without restructuring everything.
Each project gets a subdomain and its own repo. Don't retry the path approach.

## DNS

- **Registrar:** Squarespace Domains (inherited from Google Domains, which is why the
  nameservers still read `ns-cloud-*.googledomains.com`). Manage at domains.squarespace.com
  → eriksheridan.com → DNS → DNS Settings.
- **Apex** `eriksheridan.com` → A records to the four GitHub Pages IPs (185.199.108-111.153).
- **Each subdomain** → `CNAME <name> → erikrocks.github.io` (except `ebaapl`, which is a
  CNAME to a Vercel hostname).
- No CAA records. Nothing blocking Let's Encrypt.
- Erik has to add DNS records himself — there is no DNS CLI on this machine and no API
  credentials for Squarespace. Give him exact field values: Type, Host, Data.

### The certificate trap — read this before adding a subdomain

**Add the DNS record FIRST, then enable GitHub Pages.** Doing it the other way round cost
several hours on `resume.eriksheridan.com`:

GitHub requests the TLS certificate at the moment the custom domain is claimed. If the
hostname doesn't resolve yet, that request fails and **no certificate record is ever
created** — the Pages API shows `https_certificate: null` indefinitely. It does not retry
on its own. Waiting does nothing, a rebuild does nothing, and re-saving the same cname
does nothing.

The fix is to **release and re-claim the custom domain**, which forces a fresh request:

```
gh api repos/erikrocks/<repo>/pages -X PUT -f cname=""
gh api repos/erikrocks/<repo>/pages -X PUT -f cname=<host>
```

The certificate record goes `null` → `new` → `approved` within a few minutes. A transient
`status: errored` right after the re-claim is normal; a build kicks off immediately after.
Then turn on enforcement:

```
gh api repos/erikrocks/<repo>/pages -X PUT -F https_enforced=true
```

Diagnostic that actually tells you the truth — if this prints `*.github.io` rather than the
hostname, GitHub has not issued a certificate for the custom domain:

```
echo | openssl s_client -connect <host>:443 -servername <host> 2>/dev/null \
  | openssl x509 -noout -subject
```

## This page

One self-contained `index.html`. No build step, no framework, no separate assets — same
rule as KUDR. ~100KB, almost all of it the embedded headshot.

**Layout — "split rail"** (chosen by Erik from four wireframed options):

- CSS grid, `300px 1fr`, 72px gap, max-width 1000px.
- **Left rail** (`.rail`) — `position: sticky`, holds headshot, name, title, tagline, email
  and LinkedIn. Identity stays on screen while the right side scrolls.
- **Right column** — a `What I'm building` label, then three `<a class="entry">` blocks
  (EBAAPL, KUDR, Resume). The whole block is the link, not just the title. Each has a
  kicker, a serif name with an arrow, a description, and its bare address.
- **Below 800px** the grid collapses to one column and the rail goes `position: static`,
  stacking above the entries.

**Design system** — inherited from the resume so the two sites read as one:
`DM Serif Display` for names, `DM Sans` for everything else, green `#3B6D11`, ground
`#fafaf8`, borders `#e4e4de`. Light theme only, deliberately — matches the resume.

**The resume is NOT a project entry** — it was demoted on 4 Nov 2026 once there were four
real projects. It now sits in the left rail as a `.rail-link`, **between the email address
and LinkedIn**, with a document icon matching the other two. The `What I'm building` list
holds only things Erik built. Don't put the resume back in that list.

## Entry icons

Each project entry shows **that project's own app icon**, as a phone-home-screen tile in a
left gutter. They are not drawn by hand -- they are copies of the real icons, pulled from
the live sites (`icon-192.png` from each manifest; EBAAPL serves `/icon.svg`). If a project
changes its icon, re-pull it here; nothing syncs automatically.

**The source must be a full square, not a pre-rounded icon.** The hub rounds the tiles
itself; an icon with transparent rounded corners shows white wedges at the corners under
that. GameTimeTalk's 7 Nov 2026 redesign ships rounded `icon-192`/`icon-512` PNGs, so its
tile comes from the square 180px `apple-touch-icon.png` instead (still 2x+ at 48px).

| Entry | File | Source |
|---|---|---|
| GameTimeTalk | `icons/gametimetalk.png` | `gametimetalk.com/gametimetalk/apple-touch-icon.png` |
| EBAAPL | `icons/ebaapl.svg` | `ebaapl.eriksheridan.com/icon.svg` |
| KUDR | `icons/kudr.png` | `kudr.eriksheridan.com/icon-192.png` |
| Bubble | `icons/bubble.png` | `bubble.eriksheridan.com/icon-192.png` |

- 48px desktop, 42px mobile, `border-radius` ~23% (the iOS tile proportion), hairline ring
  and a soft shadow, lifting 2px on hover. Source images are 192px so they stay sharp on
  retina.
- **EBAAPL's SVG has had its own `rx="12"` stripped** so the CSS radius governs all four
  tiles identically. Keep it that way if you re-pull it.
- `alt=""` on all four, deliberately: each tile only restates the project name next to it.
  (Opposite call from the headshot, which carries real alt text.)
- The tile is pinned to the **first line of the name** with a negative `margin-top`, not
  centred: "Kitty Unicorn Dolphin Rider" wraps to two lines on a phone and a centred tile
  would float to the middle of it. The negative margin rises into the kicker's row, which
  is harmless -- the kicker is in column 2, the tile in column 1.
- Every text row is placed in column 2 so kicker, name, description and address stay flush
  with each other and only the tile hangs out to the left.

The icons are ~45KB all told, which is more than the 9KB page itself. That is fine -- they
are separate cached files fetched in parallel, not inlined bytes re-downloaded on every
visit, which was the problem with the old headshot. Bubble's is the big one (29KB): its
pixel-art dithering defeats PNG compression. A palette re-encode only helped GameTimeTalk;
the other two have too many colours to quantize losslessly.

## The headshot

`headshot.jpg` — a real 400x400 file in the repo root, served alongside the page. It was a
base64 `data:` URI inlined in the HTML until 4 Nov 2026; extracting it took the page from
101KB to **9KB** and made the file editable by hand. **Don't inline it again.**

The same file exists in `resume.eriksheridan.com`. Changing the photo still means replacing
it in both repos. It also serves as the Open Graph preview image and the apple-touch-icon.

## Favicon and link previews

`favicon.svg` — a 352-byte SVG, green `#3B6D11` with cream "ES" in a serif. Same file in
both repos. Linked as `<link rel="icon" href="favicon.svg" type="image/svg+xml">`.

Open Graph and Twitter card tags live in the `<head>` of both pages, pointing `og:image` at
`headshot.jpg`. Added 4 Nov 2026 — before that, texting someone the link produced a bare
blue link with no card. **Keep `og:url` and `og:image` absolute**; relative URLs do not work
for previews.

## games.html — Three Lives

A second standalone page in this repo: a ranker for your top 3 games per era, then an
all-time 3. Self-contained, no build, no dependencies beyond Google Fonts. Added 6 Nov 2026
from a file Erik supplied; the original is unmodified apart from the sharing work below.

**It needs no subdomain, and that is the point.** Subdomains exist here because each
project is a *different repo* — a custom domain on a project repo serves only that repo.
`games.html` lives in this repo, so a plain path works with no DNS and no certificate.
Anything else that lands in this repo is the same: just add the file.

**GitHub Pages serves extensionless paths.** `eriksheridan.com/games` resolves to
`games.html` with no redirect and no `index.html` folder needed. Don't restructure into
`games/index.html` to get a clean URL — you already have one. (The hub entry links
`games.html` explicitly; either spelling works.)

### Adding games to the catalogue — APPEND ONLY

**Game IDs are positional.** `GAME` is built as `<era-id>-<line index>`, so `8bit-42` just
means "the 43rd line of the 8-bit list". Those IDs are what `localStorage` stores and what
share links pack. **Inserting a game mid-list silently renames everyone's saved picks** —
every game after the insertion point shifts by one, and old share links decode to the
wrong titles. There is no version check that would catch it.

So: add a new game as the **last line of its era's list**, never in year order. The grid
sorts by year at render time (`viewRank()`), so an appended game still appears in its
correct year slot — source order is invisible to the user. Nothing else needs updating: the
in-app count comes from `ids.length`, and the hub blurb stopped quoting a total on
7 Nov 2026 so it can't go stale.

To check you haven't broken anything, diff old IDs against new:

```
node -e 'const fs=require("fs");const load=f=>{const E=eval("["+/const ERAS = \[([\s\S]*?)\n\];/.exec(fs.readFileSync(f,"utf8"))[1]+"]");const G={};E.forEach(e=>e.list.trim().split("\n").forEach((l,i)=>G[e.id+"-"+i]=l.split("|")[0]));return G;};const A=load("OLD.html"),B=load("games.html");let bad=0;for(const k in A)if(A[k]!==B[k]){bad++;console.log("CHANGED",k,A[k],"->",B[k]);}console.log("remapped:",bad);'
```

Era placement follows the game's **release year**, not where a respondent filed it.

### Sharing

Picks live in `localStorage` under `save-state-v3`. **That key keeps the old name on
purpose** — the app was called Save State until 7 Nov 2026. Renaming the key would make
every existing visitor's picks unreadable, which is a silent data loss, not a cosmetic
tidy-up. Leave it, and leave the `save-state-v2` fallback read beside it. Three ways out, all on the Summary tab:

- **Copy share link** packs the picks into the URL hash. Eras become their index, games
  their index within the era, write-ins ride along as `[title, year]`; the JSON is
  base64url'd. A three-era set is ~140 characters, so it survives a text message.
- **Copy as text** is the original plain-text export.
- Opening a `#s=` link shows those picks **read-only** — banner, era switcher hidden,
  `save()` short-circuits. It must never write to the viewer's `localStorage`; "Make my own
  picks" drops back to theirs. If you touch this, re-test that the viewer's own picks
  survive opening someone else's link.

**Shared write-ins are attacker-controlled text.** They go through the same `esc()` as every
other title — a link carrying a script tag renders it as literal text. Keep it that way.

The hub tile `icons/threelives.svg` is hand-drawn: three pixel hearts (the arcade lives
counter) on the app's orange `#E8762C`, full-bleed like a phone icon. Redrawn 7 Nov 2026 --
the earlier cartridge on a pale cream ground melted into the page and read as a placeholder.
Three Lives ships no icon of its own, so there is nothing upstream to re-pull.

## Deploying and verifying

Push to `main`. GitHub Pages rebuilds in roughly a minute.

**Do not trust `/pages/builds/latest` to confirm a deploy** — it reports a stale commit sha
and will look like the deploy hung when it already shipped. Check the actual bytes:

```
curl -sS -o /tmp/live.html "https://eriksheridan.com/?cb=$RANDOM"; diff /tmp/live.html index.html
```

Or wait for real content to appear:

```
until curl -s "https://eriksheridan.com/?cb=$RANDOM" | grep -q "What I'm building"; do sleep 10; done
```

## Gotchas

- **The preview pane snapshots local files as a `data:` URL.** `navigate` and
  `location.reload()` will not pick up your edits — open a fresh preview, or you'll
  screenshot stale content and think a change didn't apply. Screenshots of local files may
  be refused outright; the live URL always works.
- **Don't judge layout from a screenshot alone.** A mobile capture of this page looked
  broken (entries missing, text column too narrow) when it was completely fine — measure
  with `getBoundingClientRect()` before "fixing" anything.
- Git identity is configured globally as `Erik Sheridan <sheridanerik@gmail.com>`.
- Anything still linking to the bare `eriksheridan.com` expecting a resume now lands on the
  menu. One click away, but worth flagging if it's a job application or a profile link.

## History

- **Sep 2026** — Apex converted from resume to hub. Resume moved to its own repo and
  subdomain. HTTPS enforcement turned on for the apex (it had been off since launch).
- **15 Sep 2026** — `bubble.eriksheridan.com` added as a fourth entry, sitting between KUDR
  and the resume so the two games read together. Adding it DNS-record-first took the
  certificate from request to approved in 30 seconds, confirming the ordering rule above.
- **4 Nov 2026** — GameTimeTalk added and, on Erik's instruction, moved to the **top** of
  the list — it leads the page. Order is now GameTimeTalk, EBAAPL, KUDR, Bubble, Resume.
  Context came from the separate "GameTimeTalk (Local)" session, which owns that project.
- **4 Nov 2026** — Resume demoted out of the project list into the rail (see above), and the
  tagline replaced with "Let's make something together." on both the hub and the resume.
  The old tagline's "Product leader. Team builder." was cut because the title line directly
  above it already reads "VP of Product" — watch for that kind of stacking when editing the
  rail copy.
- **4 Nov 2026** — all entry descriptions rewritten to a **neutral register** at Erik's
  request. No first person, no origin stories ("I got annoyed with our pick site", "I had
  the idea in a dream" — both removed), no family names. Each entry now says what the thing
  is and what is interesting about it, nothing more. **Match that register when adding one.**
- **Vivian's name was removed** from the KUDR entry at the same time; it now reads "designed
  by a seven-year-old". Keep real children's names off the public site — see the standing
  rule about never putting a real child's name in code, docs, or anything published.
  The KUDR kicker still reads "Kid designed, dad approved." — Erik kept that deliberately.
- **6 Nov 2026** — `games.html` (Three Lives, named Save State until 7 Nov) added to this repo and
  linked as a fifth hub entry, with share links added to it. No subdomain: it is a path
  in this repo (see "games.html — Three Lives").
- **4 Nov 2026** — each project entry got an icon in a left gutter. First drawn by hand,
  then replaced the same day with each project's **real app icon**, pulled from the live
  sites and styled as a phone home-screen tile (see "Entry icons").
