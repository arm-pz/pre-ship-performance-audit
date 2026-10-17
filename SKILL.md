---
name: pre-ship-performance-audit
description: Audit and fix real-world web app performance before launch using four independent read-only subagent audits covering images and fonts, frontend bundles and rendering, database queries and APIs, and server, network and user-flow measurement. Use when asked to find and fix slow pages, heavy bundles, N+1 queries, missing indexes, unoptimized images or fonts, or launch-readiness performance in an application the user owns or is authorized to maintain. Requires same-condition Lighthouse baselines and post-fix medians so claims are measured, not implied by code review. Speed only - for authorization and abuse testing use pre-ship-security-red-team, for exposed secrets and source files use web-preflight.
---

# Pre-Ship Performance Audit

## Overview

Measure performance, delegate four independent audit groups covering twenty known slow-downs, consolidate findings, implement approved fixes, and re-measure under identical conditions. Use medium effort by default; run the database group at high effort. Never remove features to gain speed. This skill is about speed only — for authorization and abuse testing use `pre-ship-security-red-team`, for accidental exposure of secrets, source and admin surfaces use `web-preflight`.

## Non-negotiable rules

- Work only in the user's repository and environments they own or are authorized to maintain.
- Before editing, run a production build and launch the production-mode app (`next build && next start` for Next.js) or use an approved preview deployment.
- Capture a reproducible baseline first. Run Lighthouse three times in mobile mode on the homepage and the two or three slowest important pages; record the median of every metric. Keep device, throttling, browser, URLs, and test conditions identical for the after-measurement.
- If a production build or preview cannot be run, state the limitation and do not invent baseline numbers. Use available tests or profilers only as clearly labeled substitutes.
- Send one subagent per audit group via the Agent tool. Subagents inspect and report only; they must not edit files or change configuration. If subagents are unavailable, run groups sequentially with the same boundaries.
- Every finding names the affected file/component, route/page, evidence, likely impact, and proposed fix.
- Do not remove features to gain speed. Ask the user before removing dependencies, changing deployment regions, changing database schema/indexes, altering database settings, or installing dependencies that modify lockfiles.
- Never cache authenticated or user-specific responses in a shared/public cache.
- Do not commit, push, deploy, or change production settings without explicit authorization.
- Prefer small, measurable changes. Preserve behavior, accessibility, SEO, and error handling.

## Workflow

### Phase 1 — Establish baseline

1. Identify framework, package manager, build scripts, hosting platform, database, routes, and existing performance tooling.
2. Run the production build and start the production server, or use an approved preview URL.
3. Run mobile-mode Lighthouse three times on the homepage and the two or three slowest important routes (chosen from real measurements or user priorities).
4. Record the median for Performance score, LCP, INP or TBT (whichever is available), CLS, FCP, and Speed Index. Also capture bundle sizes/build output and relevant server/database timings where available.
5. Record exact commands, Lighthouse version/configuration, URLs, and environment. Do not compare results from materially different conditions.

### Phase 2 — Delegate four audit groups

Dispatch one Agent per group in parallel (general-purpose agents). Each prompt must include: the repo path, the read-only constraint, the requirement to name file + route + evidence + proposed fix for every finding, and the group's checklist below.

**Group 1 — Images, fonts, third-party scripts (medium effort)**
- Images served in modern formats (WebP/AVIF) and resized near rendered dimensions, with responsive srcset/sizes.
- Large remote images delivered through the framework image component (e.g. next/image) with correct dimensions and priority.
- Supabase Storage image transformations where the user's plan supports them; do not assume plan entitlements.
- Below-the-fold images/videos lazy-loaded while the actual LCP/hero image is prioritized and not lazy-loaded.
- Fonts loaded the framework-recommended self-hosted way (e.g. next/font); reduce to one or two families without changing design.
- Unused third-party scripts removed; analytics/chat widgets deferred to after load/interactivity without breaking required behavior.

**Group 2 — Frontend code and rendering (medium effort)**
- Bundle composition and largest dependencies; use an existing bundle analyzer, or propose the lowest-risk way to add one and ask first.
- Whole-library imports replaced with only the parts used (e.g. single lodash functions or native alternatives).
- Heavy below-the-fold components (charts, editors, maps) behind dynamic import.
- Homepage-critical data fetched server-side so initial HTML contains content.
- AI responses streamed as generated, preserving error handling and cancellation.
- Skeletons or stable placeholders for loading states without layout shift.
- Hydration cost, unnecessary client components, repeated renders, oversized client state.
- Identify the actual bottleneck first; do not change architecture for a theoretical score gain.

**Group 3 — Database and APIs (HIGH effort)**
- Indexes for columns used in WHERE, JOIN, ORDER BY, pagination, and RLS policies (including fields like user_id when actually used).
- Read query plans and selectivity before proposing indexes; consider composite column order, existing indexes, write overhead, storage cost, redundant indexes.
- Supabase/Postgres RLS: check whether stable expressions like `auth.uid()` can be written as `(select auth.uid())` per policy; validate plans, do not mechanically rewrite every policy.
- N+1 patterns replaced with joins, batching, or one well-shaped query.
- Long lists paginated; prefer stable cursor/keyset pagination while retaining sort semantics.
- Cache only responses identical for all users; explicitly exclude user-specific content from shared caches.
- Heavy work (email, image processing, reports) moved to background jobs/queues.

For each database recommendation include: query/policy location and affected route; observed pattern with plan/timing evidence where available; proposed index or rewrite and why it matches the workload; expected read gain and write/storage trade-offs; migration and rollback approach; tests to prove correctness and authorization are preserved. Do not create indexes, modify RLS, run migrations, or change DB settings without approval — prepare a proposed migration for review.

**Group 4 — Server, network, routing, user flows (medium effort)**
- Server functions co-located with the database; verify the actual configured region (do not assume every Vercel project is US East) and report region-mismatch latency.
- Rarely-changing routes eligible for static generation or ISR without serving stale or personalized data.
- Sequential API calls that can safely run in parallel or be consolidated.
- Main user journeys walked in Chrome mobile emulation with a low-end Android profile and slow-4G throttling.
- Slow server rendering, oversized responses, avoidable redirects, compression, cache headers, unnecessary round trips.
- Ask before changing deployment regions or production network settings; report expected latency benefit and operational trade-offs.

### Phase 3 — Consolidate findings

Merge duplicates into a prioritized backlog: P0 user-visible failure caused by performance handling; P1 major bottleneck on a core route; P2 meaningful non-critical; P3 minor or speculative. Each finding carries the `finding-id`, route, evidence reference, severity and remediation fields in Finding record below, plus priority, measured evidence, root-cause hypothesis, proposed change, expected benefit and trade-offs, approval requirement, and verification test. Mark unsupported assumptions explicitly — "best practice" alone is not proof of a bottleneck.

### Phase 4 — Implement approved fixes

- Implement highest-impact low-risk fixes first; the main agent does all edits itself.
- Keep changes grouped and reviewable; preserve semantics for authentication, authorization, RLS, personalized data, SEO, accessibility, and analytics.
- Add or update tests where behavior could regress.
- Rebuild in production mode after changes and fix build/runtime errors before benchmarking.

### Phase 5 — Measure after changes

Repeat the exact baseline procedure: same URLs, same mobile Lighthouse configuration, three runs per page, report medians, same build mode and comparable environment; note any unavoidable differences.

Deliver a before-and-after table (Page/route | Metric | Before median | After median | Change | Notes) covering Performance, LCP, INP/TBT as available, CLS, FCP, Speed Index, and bundle/transfer size when measured. Compute percentages only from valid comparable values; do not claim causality when multiple variables changed.

Deliver a fix table (Finding | File/route | Change made | Approval status | Test result | Measured impact).

### Phase 6 — Final review

- Run the production build and relevant automated tests.
- Review the full diff for behavior, accessibility, security, caching, and SEO regressions.
- Verify private responses are not publicly cached and image/font loading has no broken assets.
- List all proposed-but-unapproved changes as untouched.
- Do not commit, push, or deploy unless explicitly authorized.
- Summarize largest wins, remaining bottlenecks, test limitations, and next best action.

## Finding record

Publish these five fields for every finding so the release summary and the other pre-ship workflows can cite it:

- `finding-id` - `PERF-<nn>`; reuse the same id for the same bottleneck across reruns.
- `route` - the page, endpoint, query, or component affected, with the file.
- `evidence-ref` - the baseline median, trace, query plan, or bundle figure that shows it.
- `severity` - map the P0-P3 priority above: P0 CRITICAL, P1 HIGH, P2 MEDIUM, P3 LOW. Keep P0-P3 as the working label.
- `remediation` - `open`, `fixed-verified`, `fixed-unproven`, or `accepted-risk`; `fixed-verified` requires an after-median measured under the same conditions.

Performance findings never change the security verdicts of `web-preflight` or `pre-ship-security-red-team`. Cite those by `finding-id`.

## Completion checklist

- Baseline production build/preview captured
- Lighthouse run three times per selected page in mobile mode
- Four independent audit groups completed
- Every finding includes file and affected page
- Database index proposals evidence-based with trade-offs
- No feature removed for a score
- Approvals obtained before dependency, database, or region changes
- Fixes and regression tests completed
- Same-condition Lighthouse measurements repeated
- Before-and-after tables delivered
- Unresolved issues and measurement limitations disclosed
