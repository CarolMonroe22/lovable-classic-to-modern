# Conversion Steps (Local)

## 1. Install Dependencies

```bash
npm uninstall react-router-dom react-helmet-async next-themes lovable-tagger
npm install @tanstack/react-router @tanstack/react-start @tanstack/router-plugin vite-tsconfig-paths @vitejs/plugin-react
npm install vite@^7
```

Note: Use @vitejs/plugin-react (NOT @vitejs/plugin-react-swc). Check version compatibility with `npm view @vitejs/plugin-react@latest peerDependencies`.

## 2. Create vite.config.ts

Replace entirely:

```ts
import { defineConfig } from "vite";
import { tanstackStart } from "@tanstack/react-start/plugin/vite";
import viteReact from "@vitejs/plugin-react";
import tsconfigPaths from "vite-tsconfig-paths";

export default defineConfig({
  server: { host: "::", port: 8080 },
  plugins: [tsconfigPaths(), tanstackStart(), viteReact()],
});
```

NO nitro(). NO entry files. tanstackStart handles SSR.

## 3. Create src/router.tsx

```tsx
import { QueryClient } from "@tanstack/react-query";
import { createRouter as createTanStackRouter } from "@tanstack/react-router";
import { routeTree } from "./routeTree.gen";

export function createRouter() {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { staleTime: 60_000 } },
  });
  return createTanStackRouter({
    routeTree,
    context: { queryClient },
    scrollRestoration: true,
  });
}

export function getRouter() {
  return createRouter();
}

declare module "@tanstack/react-router" {
  interface Register {
    router: ReturnType<typeof createRouter>;
  }
}
```

QueryClient MUST be inside createRouter (per-request, not module-level).

## 4. Create src/routes/__root.tsx

```tsx
import * as React from "react";
import type { QueryClient } from "@tanstack/react-query";
import { QueryClientProvider } from "@tanstack/react-query";
import { Outlet, Link, createRootRouteWithContext, HeadContent, Scripts } from "@tanstack/react-router";
import appCss from "../index.css?url";

interface RouterContext { queryClient: QueryClient }

function RootComponent() {
  const { queryClient } = Route.useRouteContext();
  return (
    <QueryClientProvider client={queryClient}>
      {/* Add your providers here - ThemeProvider, AuthProvider, etc. */}
      <Outlet />
    </QueryClientProvider>
  );
}

function RootShell({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <head><HeadContent /></head>
      <body>{children}<Scripts /></body>
    </html>
  );
}

export const Route = createRootRouteWithContext<RouterContext>()({
  head: () => ({
    meta: [
      { charSet: "utf-8" },
      { name: "viewport", content: "width=device-width, initial-scale=1" },
      { title: "Your Site" },
    ],
    links: [
      { rel: "stylesheet", href: appCss },
      // Add font imports, favicon, etc.
    ],
  }),
  component: RootComponent,
  notFoundComponent: () => <div>404</div>,
  shellComponent: RootShell,
});
```

Use createRootRouteWithContext (NOT createRootRoute). Providers render directly (no mounted guard needed if hooks have SSR defaults).

## 5. Create File-Based Route Files

For each route in AnimatedRoutes.tsx, create a file in src/routes/:

```
/              -> src/routes/index.tsx
/blog          -> src/routes/blog/index.tsx
/blog/:slug    -> src/routes/blog/$slug.tsx
/about         -> src/routes/about.tsx
/services/foo  -> src/routes/services/foo.tsx
```

Each route file:

```tsx
import { createFileRoute } from "@tanstack/react-router";
import PageComponent from "@/pages/PageName";

export const Route = createFileRoute("/path")({
  component: PageComponent,
});
```

For routes with dynamic params:

```tsx
export const Route = createFileRoute("/blog/$slug")({
  component: PageComponent,
});
```

## 6. Replace react-router-dom Imports

In ALL files that import from react-router-dom:

| From | To |
|---|---|
| `import { Link } from "react-router-dom"` | `import { Link } from "@tanstack/react-router"` |
| `import { useNavigate } from "react-router-dom"` | `import { useNavigate } from "@tanstack/react-router"` |
| `import { useLocation } from "react-router-dom"` | `import { useLocation } from "@tanstack/react-router"` |
| `import { Navigate } from "react-router-dom"` | `import { Navigate } from "@tanstack/react-router"` |
| `navigate("/path")` | `navigate({ to: "/path" })` |
| `useParams<{ slug: string }>()` | `getRouteApi("/blog/$slug").useParams()` |

## 7. Make Context Hooks SSR-Safe

Every context hook that throws when provider is missing must return defaults instead:

```tsx
// BEFORE (breaks SSR)
export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) throw new Error("...");
  return context;
};

// AFTER (SSR-safe)
const DEFAULTS: AuthContextType = {
  user: null, profile: null, isLoading: true, isAdmin: false,
  signUp: async () => {}, signIn: async () => {}, signOut: async () => {},
};

export const useAuth = (): AuthContextType => {
  const context = useContext(AuthContext);
  return context ?? DEFAULTS;
};
```

Apply to ALL context hooks: useAuth, useTheme, useLanguage, etc.

## 8. Delete Old Files

- index.html
- src/main.tsx
- src/App.tsx
- src/components/AnimatedRoutes.tsx (or equivalent route config)
- src/components/DemflowRedirect.tsx (or similar redirect components)
- scripts/prerender.mjs (if exists)
- Any puppeteer devDependency

## 9. Update package.json Scripts

```json
{
  "dev": "vite dev",
  "build": "vite build",
  "build:dev": "vite build --mode development",
  "preview": "vite preview"
}
```

## 10. Generate Route Tree + Build

```bash
npx vite build
```

This generates src/routeTree.gen.ts automatically and builds the project. If build succeeds, test with `npx vite dev`.
