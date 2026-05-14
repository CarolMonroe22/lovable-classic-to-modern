# Lovable Classic to Modern Migration

Migrate your Lovable project from **Classic** (React + Vite + React Router) to **Modern** (TanStack Start + TanStack Router + SSR).

Your site looks and works the same after migration. What changes is how it renders - from client-only SPA to server-side rendered. Better SEO, faster first paint, proper social sharing.

> **Status: Work in progress.** The local conversion works - routes convert, code compiles, patterns are validated. But deploying to Lovable's modern stack (Cloudflare Workers) hits SSR runtime issues I haven't fully resolved yet: h3 module errors, worker bundle failures, Tailwind v3-to-v4 incompatibilities. The conversion steps, anti-patterns, and security considerations documented here are real and tested. The end-to-end deployment is not proven yet.

> **Looking for collaborators.** If you've successfully deployed a Classic-to-Modern converted project on Lovable, or if you're working on TanStack Start and want to figure this out together, I'd love to hear from you. [Open an issue](https://github.com/CarolMonroe22/lovable-classic-to-modern/issues) or find me on [LinkedIn](https://linkedin.com/in/carolmonroe).

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

**Note:** This is where my own migration stalled. The Lovable agent made good progress on SSR fixes, but the final deployment to Cloudflare Workers hit h3 module and worker bundle issues that multiple rounds of fixes didn't resolve. If you get past this point, please share what worked.

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

## Known Deployment Blockers

These are the issues I hit when deploying to Lovable's modern stack (Cloudflare Workers). Documenting them here so others can help solve them or avoid them.

| Issue | What happened | Status |
|---|---|---|
| **h3 module import errors** | SSR bundle can't resolve h3 v2 imports at runtime | Unresolved |
| **Worker bundle failures** | Cloudflare Worker fails to start after build | Unresolved |
| **Tailwind v3 to v4** | Lovable Modern uses Tailwind v4 but converted code has v3 syntax | Partially fixed (CSS migrated, runtime issues remain) |
| **Vinxi/Nitro conflicts** | Stale dependencies from conversion cause bundler conflicts | Removed but may resurface |

If you've solved any of these, [open an issue](https://github.com/CarolMonroe22/lovable-classic-to-modern/issues) with your solution.

## Help Improve This

This is actively being refined. Looking for:

- **People who've deployed Classic-to-Modern on Lovable** - your experience is the missing piece
- Testing on different Lovable Classic projects
- Edge cases and bug reports
- Security review from the community
- Feedback on clarity

[Open an issue](https://github.com/CarolMonroe22/lovable-classic-to-modern/issues) or find me on [LinkedIn](https://linkedin.com/in/carolmonroe).

## Related

- [Lovable Cloud to Supabase Migration](https://github.com/CarolMonroe22/lovable-cloud-to-supabase-migration)
- [Vibe Coding Setup](https://github.com/CarolMonroe22/vibe-coding-setup)
- [Blog post: Lovable + TanStack Start - what changed](https://carolmonroe.com/blog/lovable-tanstack-start-what-changed)
- [Blog post: Migration guide](https://carolmonroe.com/blog/lovable-to-tanstack-start-migration-guide)

## Author

**Carol Monroe** - Lovable Champion, Supabase SupaSquad

Built with Claude Code. Tested locally. Deployment in progress. Mistakes included for free.
