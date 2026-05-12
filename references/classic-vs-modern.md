# Classic vs Modern - Key Differences

## Architecture

| Aspect | Classic (SPA) | Modern (TanStack Start) |
|---|---|---|
| Rendering | Client-only. Browser downloads JS, executes, renders | Server renders HTML, browser hydrates |
| Entry point | index.html + main.tsx | __root.tsx (server-rendered shell) |
| Routing | react-router-dom (centralized in App.tsx) | @tanstack/react-router (file-based in src/routes/) |
| Data fetching | useEffect + useState (client) | createServerFn + loaders (server) |
| SEO | Requires prerendering script (puppeteer) | Built-in via SSR + head() per route |
| Auth | Entirely client-side | Client-side with _authenticated layout |
| Build output | Static files (dist/) | Server bundle + client bundle (.output/) |
| Deploy | Any static host (CDN) | Needs server runtime (Cloudflare Workers, Node.js) |

## What Changes in Code

| Code area | Classic | Modern |
|---|---|---|
| `import { Link } from` | `react-router-dom` | `@tanstack/react-router` |
| `navigate("/path")` | works | `navigate({ to: "/path" })` |
| `useParams<{slug}>()` | works | `getRouteApi("/path/$slug").useParams()` |
| Route definition | `<Route path="/blog" element={<Blog />} />` | `createFileRoute("/blog")({ component: Blog })` |
| SEO meta tags | react-helmet-async or PageSEO component | head() function in route definition |
| Protected routes | `<ProtectedRoute>` wrapper component | `_authenticated` layout route with ssr:false |
| Context hooks | Can throw if no provider | MUST return defaults (SSR safety) |
| `new Date()` in render | Works fine | Must use useClientNow hook |
| `localStorage` in useState | Works fine | Must defer to useEffect |
| QueryClient | Module-level singleton | Per-request in createRouter() |

## What Stays the Same

- All component code (just import changes)
- All Tailwind CSS / shadcn/ui styling
- All Supabase queries and integration
- All Framer Motion animations
- All static assets and images
- All data structures and types
- Design, colors, typography - everything visual

## SSR Tradeoffs

| Gain | Cost |
|---|---|
| Better SEO (HTML served to crawlers) | Context hooks need SSR defaults |
| Faster first paint (HTML arrives pre-rendered) | Hydration mismatches if server/client differ |
| Server-side data loading (single query) | Can't use window/document/localStorage in render |
| Social sharing works without edge functions | Build is slower (server + client bundles) |
| | Deploy needs server runtime, not just CDN |
| | Time-dependent UIs flash default before real value |
