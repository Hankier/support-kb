# kb — the support knowledge base

**kb** = knowledge base. What the support desk knows about how our products break, written
down where the code can keep it honest.

One desk takes reports for every product we run: `sched-prod` (shift planning, about 300k
people), `tram-3` (departure boards), `rota-6` (care-home rotas) and the rest. When something
goes wrong, the reports reach the desk first. This kb is how the desk triages them, what each
incident turned out to be, and why we decided what we decided.

## What's here

```
src/routes/kb/      the pages (.svx: Markdown with components)
  triage/           how the desk handles incoming reports
  incidents/        one page per big ticket
  decisions/        numbered decision records
src/lib/kb/         components and the logic they run
src/lib/layouts/    the frame every page shares
src/lib/data/       exports from the ticket system
```

## Conventions

- Words in the page, logic in `src/lib` (imported as `#lib/…`). A number that can be computed belongs in a component.
- Every page starts with frontmatter: `title`, `owner`, `status`, `reviewed`.
- A decision is never edited after it is accepted. A new record replaces it.
- A page you add is signed: its frontmatter carries your `signature` and it ends with `<RenderStamp {signature} />`, so the desk knows who wrote it.

## Owners

Support desk (Tove). Decision records: whoever signed them.

---

Exported from the `support` repo, `apps/kb/` → `src/`.

## This repo

The kb export as received, plus a build. `npm install && npm run build` writes every page as
HTML to `build/` (SvelteKit 3 + mdsvex, adapter-static). `build/` is committed: it is the
pages as the desk sees them.

- New page: `src/routes/kb/incidents/4471/issues/+page.svx` → `build/kb/incidents/4471/issues.html`
- Its table: `src/lib/kb/IssueTickets.svelte`, tickets in `src/lib/data/tickets-4471.json`
- The issues of #4471 are this repo's issues #1–#3.
