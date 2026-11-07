# Prompt: admin section thumbnails

Paste everything below the line into Claude Code inside a project cloned from axys-template. The reference implementation is the grovewood project at `~/Desktop/work/axys-creative/projects/grovewood`.

---

Add auto-generated section thumbnails to this project's Decap admin, copying the working implementation from `~/Desktop/work/axys-creative/projects/grovewood`.

**What it does.** When content in `src/lib/content/` is pushed to `main`, a GitHub Action builds the site, screenshots only the sections whose content file changed, and commits the JPEGs to `static/admin-thumbs/<collection>/<entry>.jpg`. The admin's grid cards show them.

**Copy these files from grovewood**, then adapt:

1. `.github/workflows/admin-thumbnails.yml`
2. `scripts/thumbnails.ts`
3. `src/routes/admin/card-thumbnails.ts`

**Wire it up:**

- Add `playwright` as a devDependency (`bun add -d playwright`) and a script `"thumbnails": "vite build && bun scripts/thumbnails.ts --all"` to `package.json`.
- In `src/routes/admin/+page.svelte`, import `cardThumbnails` from `./card-thumbnails`, call it on mount, and call its returned cleanup on destroy (match how `view-controls.ts` is wired if present).
- Make sure `config.json` collections used for pages are named `page_<slug>` (home is `page_home`) and every section file entry has a `preview_path` of `#id`. The script derives the route from the collection name and the section from the `#id`; check that the route mapping in `scripts/thumbnails.ts` (`route()`) matches this project's page URLs, and fix it if not.
- Confirm each section component actually renders its `id` on the page, so the screenshot can find it.
- Add a short "Admin thumbnails" bullet to this project's `CLAUDE.md` rules (what it is, that it is deletable: delete the workflow, script, `card-thumbnails.ts`, its call in the admin page, the `playwright` dependency and `static/admin-thumbs`).

**Constraints:**

- Keep comments minimal, per my global rules.
- The workflow must only trigger on `paths: ['src/lib/content/**']` so the bot's own thumbnail commit does not retrigger it. Keep `concurrency`, `workflow_dispatch` (runs `--all`) and `permissions: contents: write`.
- A card without a thumbnail file must stay unchanged (the admin script already does this).
- Don't run the browser or start dev servers without asking me first. You can run `bun run check` and `bun run lint`.

**Before you finish**, tell me: whether `main` has branch protection that would block the bot's push, whether the repo is private (Actions minutes), and that I should run the workflow once manually (Actions tab, Run workflow) to generate the initial full set of thumbnails.
