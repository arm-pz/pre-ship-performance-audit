# pre-ship-performance-audit

Make a web app measurably faster before you ship it. An agent skill for Claude Code, Cursor and Qoder: four independent read-only subagents audit twenty known slow-downs across images and fonts, frontend bundles and rendering, database queries and APIs, and servers and networks — then your agent fixes them and proves each fix with mobile Lighthouse medians run under identical conditions, before and after. For developers shipping a site they own or are authorized to maintain.

## Install

```
npx skills add arm-pz/pre-ship-performance-audit --skill pre-ship-performance-audit
```

Or place `SKILL.md` in your agent's skills directory, for example:

```bash
# Claude Code (project)
mkdir -p .claude/skills/pre-ship-performance-audit && cp SKILL.md .claude/skills/pre-ship-performance-audit/
# Qoder (user)
mkdir -p ~/.qoder/skills/pre-ship-performance-audit && cp SKILL.md ~/.qoder/skills/pre-ship-performance-audit/
```

This repo is also a Claude Code plugin (`.claude-plugin/plugin.json`); it is listed in the `arm-pz/caffeinity-skills` marketplace.

## Ask it things like

- `/pre-ship-performance-audit` — check this site before I deploy
- The homepage feels slow on mobile, find out why and fix it
- Run the four audits and give me before/after Lighthouse numbers
- Why is this list page slow? Check queries and indexes

## How it works

1. **Baseline** — production build (or approved preview), mobile Lighthouse ×3 per page, medians recorded. If a production build cannot run, the limitation is stated; no numbers are invented.
2. **Four read-only subagent audits** — one per group: images/fonts/third-party scripts; frontend bundle and rendering; database and APIs (query plans, indexes, RLS, N+1, pagination, caching, background jobs — run at higher effort); server/network/routing plus main user flows under low-end mobile emulation.
3. **Consolidated findings** — prioritized P0–P3 backlog; every finding names file, route, measured evidence, proposed fix, and approval status.
4. **Fixes** — applied centrally by the main agent, highest-impact low-risk first.
5. **Re-measurement** — same URLs, same configuration, ×3 runs, medians; before/after table and per-fix table.
6. **Final review** — build, tests, diff review for behavior, accessibility, security, caching and SEO regressions.

## What it will not do

- Remove features to gain a score.
- Invent baseline numbers when a comparable measurement is impossible.
- Touch dependencies, database schema/indexes, RLS policies or deployment regions without asking first.
- Cache authenticated or user-specific responses in a shared cache.
- Commit, push or deploy without explicit authorization.

## Related skills

This family covers pre-launch checks; each stays in its lane:

- [web-preflight](https://github.com/arm-pz/web-preflight) — lightweight check for accidental exposure of secrets, source and admin surfaces.
- [pre-ship-security-red-team](https://github.com/arm-pz/pre-ship-security-red-team) — authorized abuse and authorization testing: 14 checks executed with captured evidence, then fixed and rerun.

Findings cross-reference by ID (`PERF-03`, `PREFLIGHT-01`, `REDTEAM-S02`) rather than merging verdicts; each skill defines the shared five-field record in its own `SKILL.md`.
- **This skill is about speed.** Despite similar names, it is unrelated to other authors' generic "performance-audit" skills; the difference here is the four read-only subagent audits plus measured same-condition before/after medians.

## Example output

Not yet included — a real before/after medians table will be added from an actual run. No sample is shipped that the workflow never produced.

## License

MIT © 2026 Abdul Rahman P Z
