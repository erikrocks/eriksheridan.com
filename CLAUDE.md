# CLAUDE.md — eriksheridan.com

## What this is

The **apex domain**, which serves a one-page *menu* pointing at three other addresses.
It is not the resume any more — the resume moved to its own subdomain in September 2026.

Erik is not an engineer. Handle tooling, DNS and setup for him rather than handing over
commands, and explain changes by their consequence ("visitors will see X") rather than by
mechanism.

## The four addresses

| Address | Repo | Host | What it is |
|---|---|---|---|
| `eriksheridan.com` | `erikrocks/eriksheridan.com` (this one) | GitHub Pages | The menu / hub |
| `resume.eriksheridan.com` | `erikrocks/resume.eriksheridan.com` | GitHub Pages | The resume |
| `kudr.eriksheridan.com` | `erikrocks/kitty-unicorn-game` | GitHub Pages | Vivian's browser game |
| `ebaapl.eriksheridan.com` | `erikrocks/ebaapl` (private) | **Vercel** | NCAA pick'em app |

All three GitHub Pages sites: branch `main`, path `/`, custom domain set via a `CNAME`
file in the repo root, certificate approved, **Enforce HTTPS on**. Push to `main` and it
redeploys; nothing else to configure.

EBAAPL is the odd one out — Next.js on Vercel, private repo, auto-deploys on push. Its DNS
points at Vercel, not GitHub. Don't try to manage it through the Pages API.

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

**Resume is currently a peer entry**, same visual weight as the projects. Erik's plan is to
demote it to a small corner link once there are more projects. That's a small edit: pull the
third `.entry` out and add a link in a header.

## The headshot

A base64 `data:` URI embedded directly in the HTML, ~92KB, and **duplicated byte-for-byte in
both `eriksheridan.com/index.html` and `resume.eriksheridan.com/index.html`**. Changing the
photo means changing both files. There is no image file anywhere.

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
