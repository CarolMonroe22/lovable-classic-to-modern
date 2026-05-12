# Lovable Classic to Modern Migration

Migrate your Lovable project from **Classic** (React + Vite + React Router) to **Modern** (TanStack Start + TanStack Router + SSR).

Your site looks and works the same after migration. What changes is how it renders - from client-only SPA to server-side rendered. Better SEO, faster first paint, proper social sharing.

> **v0.1 - Early release.** This guide is based on real production migrations and is actively being tested across different project types. Looking for people to try it and share feedback. [Open an issue](https://github.com/CarolMonroe22/lovable-classic-to-modern/issues) with what worked and what didn't.

## Security Considerations

Moving from SPA to SSR changes your security surface. Review these before and during migration.

### Supply Chain

**TanStack NPM supply chain attack (May 2026):** 84 malicious versions were published across 42 `@tanstack/*` packages. The malware harvests credentials and persists via `.claude/` and `.vscode/` directories.

Before installing any `@tanstack/*` package:
- Verify versions against the [official TanStack postmortem](https://tanstack.com/blog/npm-supply-chain-compromise-postmortem)
- Check for the domain `filev2.getsession[.]org` in node_modules: `grep -r "getsession" node_modules/@tanstack/`
- Inspect `.claude/` and `.vscode/` for unauthorized files
- If compromised, rotate ALL credentials (AWS, GCP, GitHub, SSH)

### SSR-Specific Risks

| Risk | What happens | Mitigation |
|---|---|---|
| **QueryClient shared between requests** | User A sees User B's cached data | Create QueryClient per-request inside createRouter(), never at module level |
| **createServerFn = API endpoint** | Server functions accept external input, can be exploited without validation | Always use `.inputValidator()` on every createServerFn |
| **Environment variable exposure** | `VITE_` vars are bundled into client JS. Non-prefixed are server-only | Never prefix secrets with `VITE_`. Double-check after migration |
| **Auth state in SSR** | Server renders HTML before knowing who the user is. Private data can leak into page source | Use `_authenticated` layout with `ssr: false` for all private routes |
| **service_role in server functions** | Bypasses RLS entirely. Malicious input = full DB access | Prefer anon key + RLS in server functions. Only use service_role with strict input validation |
| **Lost security headers** | SPA _redirects/_headers (Netlify/Vercel) don't carry over to Workers | Reimplement CSP, CORS, and cache headers in the SSR layer |

## Who This Is For

- Lovable projects with their **own Supabase** (not Lovable Cloud)
- Sites that need SEO: blogs, portfolios, landing pages, content sites

**Not for:** Internal tools, dashboards, or apps with no SEO needs. SSR adds complexity. If your SPA works, evaluate before migrating.

For Lovable Cloud projects, use the [Lovable Cloud to Supabase migration](https://github.com/CarolMonroe22/lovable-cloud-to-supabase-migration) first.

## What's Inside

```
SKILL.md                              # Main guide - 5 phases
references/
  conversion-steps.md                 # 10 local conversion steps
  anti-patterns.md                    # 10 things NOT to do
  correct-patterns.md                 # 8 verified code patterns
  classic-vs-modern.md                # Comparison table
```

## The 5 Phases

1. **Audit** - routes, react-router imports, SSR-unsafe code
2. **Convert locally** - TanStack deps, file-based routes, SSR-safe hooks
3. **Push + Import** - GitHub repo, Lovable Modern project, import
4. **Lovable finishes the migration** - the Lovable agent handles platform-specific fixes
5. **Verify** - rendering, navigation, dynamic routes, view-source

### About Phase 4: Lovable Finishes the Migration

The local conversion (Phases 1-3) handles the structural work - routing, imports, file structure. But TanStack Start on Lovable has platform-specific requirements that the Lovable agent knows best.

After importing your code, ask the Lovable agent to run an audit and fix what's needed:

```
Review this project for SSR compatibility. Check for:
- Hydration mismatches (useState with Date/localStorage in initializers)
- Missing head() with unique title/description on each route
- Auth routes that need _authenticated layout with ssr:false
- Loaders that should use createServerFn instead of browser client
- shadcn components importing next-themes
- Any build errors or SSR warnings
Fix what you find.
```

The agent will typically fix hydration issues, add SEO per route, set up authenticated layouts, convert loaders to server functions, and clean up SPA leftovers.

**This phase uses Lovable credits.** Budget 100-200 credits for the full migration depending on project complexity (number of routes, dynamic content, auth flows). The agent does platform-specific optimization - hydration fixes, per-route SEO, server functions, auth layouts - that would take significantly longer to do manually.

## How to Use

**Claude Code** (recommended): clone into your skills directory.

```bash
git clone https://github.com/CarolMonroe22/lovable-classic-to-modern.git ~/.claude/skills/lovable-classic-to-modern
```

**Claude Chat**: copy SKILL.md and relevant reference files into your conversation.

**Manual**: read SKILL.md and references/conversion-steps.md for step-by-step.

## What Changes vs What Stays

| Changes | Stays the same |
|---|---|
| react-router-dom to @tanstack/react-router | All components and styling |
| index.html to server-rendered __root.tsx | Tailwind, shadcn/ui, Framer Motion |
| Client-only rendering to SSR | All Supabase queries and data |
| Centralized routes to file-based | All static assets |

## Help Improve This

This is actively being refined. Looking for:

- Testing on different Lovable Classic projects
- Edge cases and bug reports
- Security review from the community
- Feedback on clarity

[Open an issue](https://github.com/CarolMonroe22/lovable-classic-to-modern/issues) or find me on [X @carolmonroe](https://x.com/carolmonroe).

## Related

- [Lovable Cloud to Supabase Migration](https://github.com/CarolMonroe22/lovable-cloud-to-supabase-migration)
- [Vibe Coding Setup](https://github.com/CarolMonroe22/vibe-coding-setup)

## Author

**Carol Monroe** - Lovable Champion, Supabase SupaSquad

Built with Claude Code. Tested in production. Mistakes included for free.
