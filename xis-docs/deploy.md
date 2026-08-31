# Hosting these docs

This site is [docsify](https://docsify.js.org). There is no build step — `index.html` loads
docsify from a CDN and it fetches the markdown files at runtime. Edit a `.md` file, reload, done.

## Running it locally

Docsify **cannot** run from `file://`. Opening `index.html` by double-clicking it shows a blank
page, because the browser blocks the requests it makes for the markdown. You need a server:

```bash
npx serve .
```

Or with docsify's own CLI, which adds live reload:

```bash
npm i -g docsify-cli
docsify serve .
```

Then open <http://localhost:3000>.

## Choosing a host

!> **These docs describe how a security system decides to remove players.** Even at
category level, hosting them publicly hands attackers a starting point, and any page you add
later with thresholds or detector ids on it turns a public site into a bypass guide. Pick a
host with access control, not one that is merely obscure.

| Option | Private? | Cost | Notes |
|---|---|---|---|
| **Cloudflare Pages + Access** | Yes, properly | Free | Best free option. Access is email-gated; the free Zero Trust tier covers up to 50 users. |
| **Netlify + password protection** | Yes | Paid | Site-wide password needs a paid plan. Netlify Identity is the free-ish alternative but is more setup. |
| **`docsify serve` on your own machine** | Yes | Free | No hosting at all. Fine if only you read them. |
| **GitHub Pages** | **No** | Free | See below. |
| **Vercel** | Deployment protection | Free tier limited | Password protection is a paid feature. |

### The GitHub Pages trap

A **private repository does not give you a private site.** On Free, Pro and Team plans,
enabling Pages on a private repo publishes a publicly reachable site — the repo stays private,
the site does not. Access-controlled Pages requires GitHub Enterprise Cloud.

If you host on Pages anyway, remember the URL is guessable and gets crawled.

?> `index.html` already sends `<meta name="robots" content="noindex, nofollow">`. That keeps the
site out of search results for crawlers that honour it. It is not access control and should not
be mistaken for it.

### Cloudflare Pages, start to finish

```bash
cd /c/Users/xaros/xis-docs
git init
git add -A
git commit -m "XIS AntiCheat documentation"
```

Push to a repo, then in the Cloudflare dashboard:

1. **Workers & Pages → Create → Pages → Connect to Git**, pick the repo.
2. Build command: **leave empty**. Output directory: `/`. There is nothing to build.
3. Deploy.
4. **Zero Trust → Access → Applications → Add a self-hosted application**, point it at the
   `pages.dev` hostname, and add a policy allowing the specific email addresses that should get
   in.

Step 4 is the one that matters. Without it the site is public.

## The `.nojekyll` file

Already present. GitHub Pages runs Jekyll by default, which ignores files starting with an
underscore — that would silently break `_sidebar.md` and `_coverpage.md`, and the failure looks
like "my sidebar just doesn't appear". `.nojekyll` turns Jekyll off. Harmless on every other
host.

## Adding a page

1. Create `newpage.md`.
2. Add a line to `_sidebar.md`:

```markdown
- **Reference**
  - [My new page](newpage.md)
```

That is the whole process. Headings inside the page become sub-entries automatically, up to
`subMaxLevel: 3`.

## Things worth knowing about this setup

- **Callouts** use docsify's syntax: `?>` for a tip, `!>` for a warning. Both are styled in
  `custom.css`.
- **Lua highlighting** is loaded via Prism. Use ` ```lua ` fences.
- **Search** is client-side and indexes every page; it caches for 24 hours, so a reader may need
  a hard reload to see new pages in results.
- **Dark mode** follows the reader's system setting via `prefers-color-scheme` in `custom.css`.
  There is no toggle.
- **`.gitignore`** excludes `internal/` and `NOTES.md`. Keep the full detector reference in
  there, or better, out of the repo entirely.
