# Correct Patterns (Verified May 2026)

## vite.config.ts

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

## router.tsx (QueryClient per-request)

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

## __root.tsx (createRootRouteWithContext)

```tsx
import type { QueryClient } from "@tanstack/react-query";
import { QueryClientProvider } from "@tanstack/react-query";
import { Outlet, createRootRouteWithContext, HeadContent, Scripts } from "@tanstack/react-router";
import appCss from "../index.css?url";

interface RouterContext { queryClient: QueryClient }

function RootComponent() {
  const { queryClient } = Route.useRouteContext();
  return (
    <QueryClientProvider client={queryClient}>
      {/* Providers render directly - no mounted guard */}
      <ThemeProvider>
        <AuthProvider>
          <Outlet />
        </AuthProvider>
      </ThemeProvider>
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
      { title: "Site Name" },
      { name: "description", content: "Site description" },
    ],
    links: [{ rel: "stylesheet", href: appCss }],
  }),
  component: RootComponent,
  errorComponent: RootErrorComponent,
  notFoundComponent: NotFoundComponent,
  shellComponent: RootShell,
});
```

## Dynamic Route with Loader + SEO Head

```tsx
// src/routes/blog/$slug.tsx
import { createFileRoute } from "@tanstack/react-router";
import PageComponent from "@/pages/BlogPost";
import { getBlogPostMeta } from "@/lib/posts.functions";

const SITE = "https://yoursite.com";

export const Route = createFileRoute("/blog/$slug")({
  loader: ({ params }) => getBlogPostMeta({ data: { slug: params.slug } }),
  head: ({ loaderData }) => {
    const post = loaderData;
    if (!post) return { meta: [{ title: "Post Not Found" }] };
    return {
      meta: [
        { title: `${post.title} - Site Name` },
        { name: "description", content: post.excerpt ?? "" },
        { property: "og:title", content: post.title },
        { property: "og:type", content: "article" },
        { name: "twitter:card", content: "summary_large_image" },
      ],
      links: [{ rel: "canonical", href: `${SITE}/blog/${post.slug}` }],
    };
  },
  component: PageComponent,
});
```

## createServerFn (Server-Side Data Fetching)

```tsx
// src/lib/posts.functions.ts
import { createServerFn } from "@tanstack/react-start";
import { notFound } from "@tanstack/react-router";
import { supabase } from "@/integrations/supabase/client";

export const getBlogPostMeta = createServerFn({ method: "GET" })
  .inputValidator((input: { slug: string }) => input)
  .handler(async ({ data }) => {
    const { data: post, error } = await supabase
      .from("blog_posts")
      .select("title, excerpt, featured_image, published_at, slug")
      .eq("slug", data.slug)
      .eq("is_published", true)
      .maybeSingle();

    if (error || !post) throw notFound();
    return post;
  });
```

## _authenticated Layout (Protected Routes)

```tsx
// src/routes/_authenticated.tsx
import { createFileRoute, Outlet, Navigate } from "@tanstack/react-router";
import { useAuth } from "@/contexts/AuthContext";

export const Route = createFileRoute("/_authenticated")({
  ssr: false,
  component: AuthenticatedLayout,
});

function AuthenticatedLayout() {
  const { user, isLoading } = useAuth();
  if (isLoading) return <div>Loading...</div>;
  if (!user) return <Navigate to="/auth" />;
  return <Outlet />;
}
```

Protected routes go under `_authenticated/`:
```
src/routes/_authenticated/dashboard.tsx
src/routes/_authenticated/admin/blog.tsx
```

## SSR-Safe Context Hook

```tsx
const DEFAULTS: ThemeContextType = {
  theme: "system",
  setTheme: () => {},
  resolvedTheme: "light",
};

export const useTheme = (): ThemeContextType => {
  const context = useContext(ThemeContext);
  return context ?? DEFAULTS;
};
```

## useClientNow Hook (SSR-Safe Dates)

```tsx
import { useState, useEffect } from "react";

export function useClientNow(refreshMs = 60_000): Date {
  const [now, setNow] = useState<Date>(() => new Date("2025-01-01T00:00:00Z"));

  useEffect(() => {
    setNow(new Date());
    if (!refreshMs) return;
    const id = setInterval(() => setNow(new Date()), refreshMs);
    return () => clearInterval(id);
  }, [refreshMs]);

  return now;
}
```

## Static Route with SEO

```tsx
// src/routes/about.tsx
import { createFileRoute } from "@tanstack/react-router";
import PageComponent from "@/pages/About";

export const Route = createFileRoute("/about")({
  head: () => ({
    meta: [
      { title: "About - Site Name" },
      { name: "description", content: "About page description" },
      { property: "og:title", content: "About - Site Name" },
    ],
  }),
  component: PageComponent,
});
```

## Private Route with noindex

```tsx
// src/routes/_authenticated/dashboard.tsx
export const Route = createFileRoute("/_authenticated/dashboard")({
  head: () => ({
    meta: [
      { title: "Dashboard" },
      { name: "robots", content: "noindex,nofollow" },
    ],
  }),
  component: DashboardPage,
});
```
