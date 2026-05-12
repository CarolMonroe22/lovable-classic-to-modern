---
name: lovable-classic-to-modern
description: Migrate Lovable Classic (React + Vite + React Router) projects to Lovable Modern (TanStack Start + TanStack Router + SSR). Use when users ask about converting to TanStack Start, switching from classic to modern stack in Lovable, adding SSR to a Lovable project, improving SEO with server-side rendering, migrating from React Router to TanStack Router, or converting a Lovable SPA to SSR. Scope is projects with their own Supabase (NOT Lovable Cloud - use lovable-cloud-migration skill for that).
---

# Lovable Classic to Modern Migration

> Created by **Carol Monroe** - Lovable Champion and Supabase SupaSquad Member
> Tested on 2026-05-11 with full production migration (carolmonroe.com)

## What This Does

Migrate a Lovable Classic project (React + Vite + React Router SPA) to Lovable Modern (TanStack Start + TanStack Router with SSR). The site looks and works identical after migration - only the routing layer and rendering model change.

## When to Migrate

**Yes:** SEO-dependent sites (blogs, landing pages, portfolios), social sharing needs server-rendered OG tags, need createServerFn for server-side data fetching.

**No:** Internal tools/dashboards, heavy client-only features (time-based UIs, localStorage-driven state), "because SSR is better" (it's a tradeoff).

## Prerequisites

| Tool | Required |
|---|---|
| Claude Code (Pro/Max) | Yes |
| Lovable MCP | Yes |
| GitHub CLI (`gh`) | Yes |
| Node.js 18+ | Yes |

## Migration Flow (5 Phases)

### Phase 1: Audit

Scan the source project:

```bash
grep -rl "react-router-dom" src/ --include="*.tsx" --include="*.ts" | wc -l
grep -rn "window\.\|document\.\|localStorage" src/ --include="*.tsx" --include="*.ts" | wc -l
```

Map all routes from AnimatedRoutes.tsx or App.tsx. Note dynamic routes (:slug params) and protected routes.

### Phase 2: Convert Locally

**CRITICAL: Do ALL conversion locally, NOT via Lovable messages.** Messages timeout on large transfers.

1. Copy project: `cp -r ~/project ~/project-tanstack`
2. Branch: `git checkout -b tanstack-migration`
3. Follow [references/conversion-steps.md](references/conversion-steps.md) for detailed steps
4. Build: `npx vite build`
5. Test: `npx vite dev`

### Phase 3: Push + Import

1. `gh repo create {user}/{name} --private`
2. `git push -u origin tanstack-migration`
3. Create Lovable modern project via MCP with `tech_stack: "modern"`
4. Connect GitHub in Lovable editor (Settings > GitHub)
5. Import from repo (may need temp public visibility)

### Phase 4: Platform Fixes

Tell the Lovable agent to fix SSR issues. It handles: hydration mismatches, SEO head() per route, _authenticated layout, createServerFn loaders, next-themes removal from shadcn.

### Phase 5: Verify

Check: preview renders, navigation works, dynamic routes load, protected routes redirect, view-source shows SSR HTML.

## Anti-Patterns

See [references/anti-patterns.md](references/anti-patterns.md) for full list. Critical ones:

- NO manual entry-client.tsx / entry-server.tsx (auto-generated)
- NO nitro() plugin (tanstackStart handles SSR)
- NO QueryClient at module level (create per-request in createRouter)
- NO useParams casts (use getRouteApi)
- NO bulk code via Lovable messages (use GitHub)

## Correct Patterns

See [references/correct-patterns.md](references/correct-patterns.md) for code examples of:

- vite.config.ts
- router.tsx with per-request QueryClient
- __root.tsx with createRootRouteWithContext
- File-based route with loader + head()
- createServerFn for server-side queries
- _authenticated layout route
- SSR-safe context hooks
- useClientNow hook for dates
