# AGENTS.md — khaled-portfolio

Portfolio site for Khaled Ahmed (single-page static, one `index.html`).
Lives at `https://khaled-portfolio-two.vercel.app` (Vercel, auto-deploys from
this repo's `main`) and `https://khaled2092002-alt.github.io/khaled-portfolio/`
(GitHub Pages).

## Comments system
- Section `#comments` in the HTML, rendered by `renderComments()` in the main
  script (`applyLang()` calls it on every language switch).
- Storage backend is a Cloudflare Worker (see
  `Downloads\portfolio-comments-worker\AGENTS.md`). The frontend calls it via
  two consts in the script:
  - `COMMENTS_API = "https://portfolio-comments.khaled2092002.workers.dev"`
  - `COMMENTS_PAGE = "khaled-portfolio"` (fixed page key inside KV)
- API: `GET ?page=` returns `{ok, comments}`; `POST {page,name,content,honey}`
  adds a comment (max 60/500 chars, honeypot field); `DELETE ?page=&id=`
  removes one. All responses are CORS-open.
- Delete moderation (owner only): open browser console and run
  `localStorage.setItem("cf-admin","1")`, then reload — a ✕ delete button
  appears on every comment in-UI.
- After a Worker redeploy that changes the URL, update `COMMENTS_API`.

## i18n
- Two dictionaries per language (`ar`, `en`); component texts are keys in each
  dict. Arabic is the default site language.
- Comment-related keys: `cmT`, `cmS`, `cmNeed`, `cmNamePh`, `cmContentPh`,
  `cmSend`, `cmSending`, `cmSent`, `cmFail`, `cmLoading`, `cmEmpty`, `cmErr`.
- `D()` returns the dictionary for the current language; `T()` / applyLang
  handle re-rendering. When adding UI strings: add them to BOTH `ar` and `en`
  dicts or the site throws on that language.

## Conventions
- Design tokens: `--ink`, `--paper`, `--ink2`, display font `--display`.
  Sections are `<section class="sec"><div class="wrap"><h2 class="h2">...`.
- CSS is inside `<style>` at the top of `index.html`; JS is one big IIFE at the
  bottom (starts at the first `(() => {` line). Style modifications should
  match the existing light theme (paper cards, 1.5px ink borders, offset
  shadows, 16–32px radii).

## Git / deployment
- There is NO global git identity on this machine. Always pass it inline:
  `git -C <repo> -c user.name="Khaled Ahmed" -c user.email="khaled2092002@gmail.com" commit -m "..."`
- Remote: `https://github.com/khaled2092002-alt/khaled-portfolio.git`, branch
  `main`. The local clone's `origin` is set up with the token embedded, so a
  plain `git push` works without re-pasting credentials. The token is stored
  ONLY in the local clone's config + in
  `Downloads\portfolio-comments-worker\.deploy-env.ps1` — never in committed
  files, never paste in chat. GH_TOKEN (from dot-sourcing that script) holds
  the same value if a fresh push URL needs building.
- Vercel builds from `main`, so a successful push updates the live site within
  ~1 minute.

## Verify after JS edits
- Extract the main script (`from the first \`(() => {` to its closing
  `</script>`) to a temp `.js` file and run `node --check` on it. Node v22 is
  installed.