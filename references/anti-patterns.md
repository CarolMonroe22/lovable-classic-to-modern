# Anti-Patterns - What NOT To Do

## 1. Manual Entry Files
**Don't:** Create `entry-client.tsx` or `entry-server.tsx` manually.
**Why:** TanStack Start v1.167+ auto-generates them via the vite plugin. Manual ones conflict and break the build.
**Fix:** Delete them if present.

## 2. nitro() Plugin
**Don't:** Add `nitro()` from `nitro/vite` alongside `tanstackStart()`.
**Why:** `tanstackStart()` already handles SSR. Two SSR engines create virtual entry conflicts.
**Fix:** Only use `tanstackStart()` + `viteReact()` + `tsconfigPaths()`.

## 3. Module-Level QueryClient
**Don't:** `const queryClient = new QueryClient()` at module level in __root.tsx.
**Why:** In SSR, the server handles ALL users. A shared QueryClient leaks cached data between requests.
**Fix:** Create QueryClient inside `createRouter()` - one fresh instance per request.

## 4. useParams with Unsafe Casts
**Don't:** `useParams({ strict: false }) as { slug: string }`
**Why:** No type safety. If the route changes, TypeScript won't catch the error.
**Fix:** `const routeApi = getRouteApi("/blog/$slug"); const { slug } = routeApi.useParams();`

## 5. Mounted Guard for Providers
**Don't:** Wrap providers in `{mounted && <Providers>}` pattern.
**Why:** Causes a hydration flash where the page renders without providers, then re-renders with them.
**Fix:** Make all context hooks return safe defaults when provider is missing: `return context ?? DEFAULTS;`

## 6. Bulk Code via Lovable Messages
**Don't:** Send 200+ files as code blocks in Lovable send_message.
**Why:** MCP drops on large messages (>500 lines). Timeouts. Burns credits.
**Fix:** Push to GitHub, import via Lovable's GitHub integration.

## 7. Sharing Root head() for All Routes
**Don't:** Only put head() in __root.tsx and skip individual routes.
**Why:** All 48 pages get the same `<title>` and meta tags. Terrible for SEO.
**Fix:** Each route file gets its own head() with unique title, description, OG tags.

## 8. Using @vitejs/plugin-react-swc
**Don't:** Use `@vitejs/plugin-react-swc` with TanStack Start.
**Why:** Not compatible. TanStack Start requires the standard React plugin.
**Fix:** `npm install @vitejs/plugin-react` (check version compatibility with your Vite version).

## 9. Keeping SPA Artifacts
**Don't:** Leave `scripts/prerender.mjs`, `puppeteer`, `_redirects`, `_headers` in the project.
**Why:** Dead code. SSR replaces prerendering. _redirects/_headers are for static hosting (Netlify/Vercel), not Workers.
**Fix:** Delete them and remove puppeteer from devDependencies.

## 10. useState with Date/localStorage in Initializer
**Don't:** `useState(() => new Date())` or `useState(() => localStorage.getItem("theme"))`.
**Why:** Server renders "afternoon", client renders "evening" (different timezone). Hydration mismatch.
**Fix:** `useState("afternoon")` + `useEffect(() => setMode(getAutoMode()))`. Stable default, real value after mount.
