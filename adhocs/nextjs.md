# Next.js

Next.js is a React framework for building full-stack web applications. It provides a powerful set of features including file-system based routing, server-side rendering, static site generation, and API routes. The framework leverages React Server Components for optimal performance, allowing developers to build hybrid applications that combine static and dynamic rendering strategies. Next.js supports both the modern App Router (recommended) and the legacy Pages Router.

The framework offers built-in optimizations for images, fonts, and scripts, along with features like automatic code splitting, prefetching, and caching. Next.js 16 introduces Turbopack as the default bundler, Cache Components for fine-grained caching control, and stable support for the React Compiler. The platform seamlessly integrates with Vercel for deployment but also supports self-hosting with Node.js, Docker, or static exports.

## Installation

Create a new Next.js application using the CLI tool, which sets up TypeScript, ESLint, Tailwind CSS, and the App Router by default.

```bash
# Create new app with recommended defaults
npx create-next-app@latest my-app --yes
cd my-app
npm run dev

# Or manually install dependencies
npm install next@latest react@latest react-dom@latest
```

## Project Structure

Next.js uses file-system based routing where the folder structure determines your application routes.

```
my-app/
├── app/                    # App Router (recommended)
│   ├── layout.tsx          # Root layout (required)
│   ├── page.tsx            # Home page (/)
│   ├── about/
│   │   └── page.tsx        # About page (/about)
│   ├── blog/
│   │   ├── page.tsx        # Blog list (/blog)
│   │   └── [slug]/
│   │       └── page.tsx    # Dynamic blog post (/blog/my-post)
│   └── api/
│       └── route.ts        # API route (/api)
├── public/                 # Static assets
├── next.config.ts          # Next.js configuration
└── package.json
```

## Root Layout

The root layout is required and must contain html and body tags. It wraps all pages in your application.

```tsx
// app/layout.tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}
```

## Pages

Pages are UI components that are unique to a route. Export a default component from a page.tsx file.

```tsx
// app/page.tsx
export default function HomePage() {
  return <h1>Welcome to Next.js!</h1>
}

// app/dashboard/page.tsx
export default async function DashboardPage() {
  const data = await fetchDashboardData()
  return (
    <main>
      <h1>Dashboard</h1>
      <p>Total users: {data.userCount}</p>
    </main>
  )
}
```

## Link Component

The Link component enables client-side navigation with automatic prefetching for improved performance.

```tsx
import Link from 'next/link'

export default function Navigation() {
  return (
    <nav>
      {/* Basic navigation */}
      <Link href="/dashboard">Dashboard</Link>

      {/* With query parameters */}
      <Link
        href={{
          pathname: '/search',
          query: { q: 'nextjs', page: 1 },
        }}
      >
        Search
      </Link>

      {/* Replace history instead of push */}
      <Link href="/settings" replace>
        Settings
      </Link>

      {/* Disable scroll to top */}
      <Link href="/products#section" scroll={false}>
        Products
      </Link>

      {/* Disable prefetching */}
      <Link href="/large-page" prefetch={false}>
        Large Page
      </Link>

      {/* Navigation event handler */}
      <Link
        href="/checkout"
        onNavigate={(e) => {
          if (!confirm('Leave current page?')) {
            e.preventDefault()
          }
        }}
      >
        Checkout
      </Link>
    </nav>
  )
}
```

## useRouter Hook

The useRouter hook enables programmatic navigation in Client Components.

```tsx
'use client'

import { useRouter } from 'next/navigation'

export default function NavigationButtons() {
  const router = useRouter()

  return (
    <div>
      {/* Navigate to a new page */}
      <button onClick={() => router.push('/dashboard')}>
        Go to Dashboard
      </button>

      {/* Replace current history entry */}
      <button onClick={() => router.replace('/login')}>
        Go to Login
      </button>

      {/* Navigate without scrolling to top */}
      <button onClick={() => router.push('/products', { scroll: false })}>
        Browse Products
      </button>

      {/* Refresh current route data */}
      <button onClick={() => router.refresh()}>
        Refresh Data
      </button>

      {/* Browser history navigation */}
      <button onClick={() => router.back()}>Back</button>
      <button onClick={() => router.forward()}>Forward</button>

      {/* Prefetch a route */}
      <button
        onMouseEnter={() => router.prefetch('/heavy-page')}
        onClick={() => router.push('/heavy-page')}
      >
        Heavy Page
      </button>
    </div>
  )
}
```

## Image Component

The Image component extends HTML img with automatic optimization, lazy loading, and responsive sizing.

```tsx
import Image from 'next/image'
import profilePic from './profile.png'

export default function Gallery() {
  return (
    <div>
      {/* Static import - dimensions inferred automatically */}
      <Image
        src={profilePic}
        alt="Profile picture"
        placeholder="blur"
      />

      {/* Remote image - dimensions required */}
      <Image
        src="https://example.com/photo.jpg"
        alt="Remote photo"
        width={800}
        height={600}
        quality={80}
      />

      {/* Fill parent container */}
      <div style={{ position: 'relative', width: '100%', height: 400 }}>
        <Image
          src="/hero.jpg"
          alt="Hero image"
          fill
          style={{ objectFit: 'cover' }}
          sizes="100vw"
          preload
        />
      </div>

      {/* Responsive image */}
      <Image
        src="/banner.jpg"
        alt="Banner"
        width={1200}
        height={400}
        sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
        style={{ width: '100%', height: 'auto' }}
      />

      {/* Custom loader */}
      <Image
        loader={({ src, width, quality }) =>
          `https://cdn.example.com/${src}?w=${width}&q=${quality || 75}`
        }
        src="photo.jpg"
        alt="CDN image"
        width={500}
        height={300}
      />
    </div>
  )
}

// next.config.js - Configure remote image patterns
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'example.com',
        pathname: '/images/**',
      },
    ],
  },
}
```

## Data Fetching in Server Components

Server Components can fetch data directly using async/await without useEffect or useState.

```tsx
// app/posts/page.tsx
interface Post {
  id: number
  title: string
  content: string
}

async function getPosts(): Promise<Post[]> {
  const res = await fetch('https://api.example.com/posts', {
    next: { revalidate: 3600 }, // Revalidate every hour
  })

  if (!res.ok) {
    throw new Error('Failed to fetch posts')
  }

  return res.json()
}

export default async function PostsPage() {
  const posts = await getPosts()

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.content}</p>
        </li>
      ))}
    </ul>
  )
}

// Fetch with cache tags for on-demand revalidation
async function getProduct(id: string) {
  const res = await fetch(`https://api.example.com/products/${id}`, {
    next: { tags: ['products', `product-${id}`] },
  })
  return res.json()
}

// Fetch without caching
async function getLatestNews() {
  const res = await fetch('https://api.example.com/news', {
    cache: 'no-store',
  })
  return res.json()
}
```

## Route Handlers (API Routes)

Create serverless API endpoints using Route Handlers with standard Web Request and Response APIs.

```tsx
// app/api/users/route.ts
import { NextResponse } from 'next/server'

// GET /api/users
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const limit = searchParams.get('limit') || '10'

  const users = await db.users.findMany({ take: parseInt(limit) })

  return NextResponse.json(users)
}

// POST /api/users
export async function POST(request: Request) {
  const body = await request.json()

  const user = await db.users.create({
    data: { name: body.name, email: body.email },
  })

  return NextResponse.json(user, { status: 201 })
}

// app/api/users/[id]/route.ts
export async function GET(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params
  const user = await db.users.findUnique({ where: { id } })

  if (!user) {
    return NextResponse.json(
      { error: 'User not found' },
      { status: 404 }
    )
  }

  return NextResponse.json(user)
}

export async function DELETE(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params
  await db.users.delete({ where: { id } })

  return new Response(null, { status: 204 })
}
```

## Server Actions

Server Actions allow you to define server-side functions that can be called from Client Components for data mutations.

```tsx
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string
  const content = formData.get('content') as string

  await db.posts.create({
    data: { title, content },
  })

  revalidatePath('/posts')
  redirect('/posts')
}

export async function updatePost(id: string, formData: FormData) {
  const title = formData.get('title') as string

  await db.posts.update({
    where: { id },
    data: { title },
  })

  revalidatePath(`/posts/${id}`)
}

export async function deletePost(id: string) {
  await db.posts.delete({ where: { id } })
  revalidatePath('/posts')
}

// app/posts/new/page.tsx
import { createPost } from '../actions'

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input type="text" name="title" placeholder="Title" required />
      <textarea name="content" placeholder="Content" required />
      <button type="submit">Create Post</button>
    </form>
  )
}

// Client Component with Server Action
'use client'

import { deletePost } from '../actions'
import { useTransition } from 'react'

export function DeleteButton({ postId }: { postId: string }) {
  const [isPending, startTransition] = useTransition()

  return (
    <button
      disabled={isPending}
      onClick={() => {
        startTransition(async () => {
          await deletePost(postId)
        })
      }}
    >
      {isPending ? 'Deleting...' : 'Delete'}
    </button>
  )
}
```

## Cookies and Headers

Access request cookies and headers in Server Components using async functions.

```tsx
// app/page.tsx
import { cookies, headers } from 'next/headers'

export default async function Page() {
  // Read cookies
  const cookieStore = await cookies()
  const theme = cookieStore.get('theme')?.value || 'light'
  const session = cookieStore.get('session')

  // Read headers
  const headersList = await headers()
  const userAgent = headersList.get('user-agent')
  const authorization = headersList.get('authorization')

  return (
    <div className={theme}>
      <p>User Agent: {userAgent}</p>
      {session && <p>Logged in</p>}
    </div>
  )
}

// Set cookies in Server Actions
'use server'

import { cookies } from 'next/headers'

export async function setTheme(theme: string) {
  const cookieStore = await cookies()

  cookieStore.set('theme', theme, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 60 * 60 * 24 * 365, // 1 year
  })
}

export async function logout() {
  const cookieStore = await cookies()
  cookieStore.delete('session')
}
```

## Dynamic Routes

Create dynamic route segments using brackets in folder names.

```tsx
// app/blog/[slug]/page.tsx
export default async function BlogPost({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  const post = await getPost(slug)

  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  )
}

// Generate static paths at build time
export async function generateStaticParams() {
  const posts = await getPosts()

  return posts.map((post) => ({
    slug: post.slug,
  }))
}

// app/products/[...slug]/page.tsx - Catch-all segments
export default async function ProductPage({
  params,
}: {
  params: Promise<{ slug: string[] }>
}) {
  const { slug } = await params
  // /products/a/b/c -> slug = ['a', 'b', 'c']

  return <div>Product: {slug.join('/')}</div>
}

// app/shop/[[...slug]]/page.tsx - Optional catch-all
// Matches /shop, /shop/a, /shop/a/b, etc.
export default async function ShopPage({
  params,
}: {
  params: Promise<{ slug?: string[] }>
}) {
  const { slug } = await params

  if (!slug) {
    return <div>Shop Home</div>
  }

  return <div>Category: {slug.join('/')}</div>
}
```

## Cache and Revalidation

Control caching behavior with revalidatePath, revalidateTag, and the use cache directive.

```tsx
// Revalidate specific path
'use server'

import { revalidatePath, revalidateTag } from 'next/cache'

export async function updateProduct(id: string, data: ProductData) {
  await db.products.update({ where: { id }, data })

  // Revalidate specific page
  revalidatePath(`/products/${id}`)

  // Revalidate all pages matching pattern
  revalidatePath('/products/[id]', 'page')

  // Revalidate layout and all nested pages
  revalidatePath('/products', 'layout')

  // Revalidate by cache tag
  revalidateTag('products')
  revalidateTag(`product-${id}`)
}

// Use cache directive for fine-grained caching
// Requires: cacheComponents: true in next.config.ts
import { cacheLife, cacheTag } from 'next/cache'

async function getProducts() {
  'use cache'
  cacheTag('products')
  cacheLife('hours') // Use built-in profile

  return await db.products.findMany()
}

async function getExpensiveData() {
  'use cache'
  cacheLife({
    stale: 300,      // 5 minutes client-side
    revalidate: 900, // 15 minutes server-side
    expire: 3600,    // 1 hour max age
  })

  return await computeExpensiveOperation()
}

// Cache entire page
// app/static-page/page.tsx
'use cache'

export default async function StaticPage() {
  const data = await fetchData()
  return <div>{data}</div>
}
```

## Redirect and Navigation

Redirect users programmatically in Server Components, Route Handlers, and Server Actions.

```tsx
import { redirect, permanentRedirect, notFound } from 'next/navigation'

// Server Component
export default async function ProtectedPage() {
  const session = await getSession()

  if (!session) {
    redirect('/login') // 307 temporary redirect
  }

  if (session.role !== 'admin') {
    redirect('/unauthorized')
  }

  return <AdminDashboard />
}

// Handle missing resources
export default async function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const product = await getProduct(id)

  if (!product) {
    notFound() // Renders not-found.tsx
  }

  return <ProductDetails product={product} />
}

// Server Action with redirect
'use server'

export async function createOrder(formData: FormData) {
  const order = await db.orders.create({
    data: { items: formData.getAll('items') },
  })

  revalidatePath('/orders')
  redirect(`/orders/${order.id}/confirmation`)
}

// Permanent redirect for SEO
export default async function OldPage() {
  permanentRedirect('/new-page') // 308 permanent redirect
}
```

## Error Handling

Handle errors gracefully with error.tsx boundary files.

```tsx
// app/error.tsx
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={() => reset()}>Try again</button>
    </div>
  )
}

// app/global-error.tsx - Handles root layout errors
'use client'

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <html>
      <body>
        <h2>Something went wrong!</h2>
        <button onClick={() => reset()}>Try again</button>
      </body>
    </html>
  )
}

// app/not-found.tsx
export default function NotFound() {
  return (
    <div>
      <h2>404 - Page Not Found</h2>
      <p>The page you're looking for doesn't exist.</p>
    </div>
  )
}
```

## Loading States

Show loading UI while route segments load using loading.tsx files.

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return (
    <div className="animate-pulse">
      <div className="h-8 bg-gray-200 rounded w-1/4 mb-4" />
      <div className="h-4 bg-gray-200 rounded w-1/2 mb-2" />
      <div className="h-4 bg-gray-200 rounded w-3/4" />
    </div>
  )
}

// Use Suspense for granular loading states
import { Suspense } from 'react'

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>

      <Suspense fallback={<p>Loading stats...</p>}>
        <Stats />
      </Suspense>

      <Suspense fallback={<p>Loading chart...</p>}>
        <Chart />
      </Suspense>
    </div>
  )
}

async function Stats() {
  const stats = await fetchStats() // Streams when ready
  return <StatsDisplay data={stats} />
}
```

## Proxy (Middleware)

Run code before requests are completed for authentication, redirects, and request modification.

```tsx
// proxy.ts (at project root)
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function proxy(request: NextRequest) {
  const { pathname } = request.nextUrl

  // Authentication check
  const token = request.cookies.get('auth-token')
  if (pathname.startsWith('/dashboard') && !token) {
    return NextResponse.redirect(new URL('/login', request.url))
  }

  // Add custom headers
  const response = NextResponse.next()
  response.headers.set('x-custom-header', 'my-value')

  // Rewrite URL (internal proxy)
  if (pathname.startsWith('/api/v1')) {
    return NextResponse.rewrite(
      new URL(pathname.replace('/api/v1', '/api/v2'), request.url)
    )
  }

  // Geo-based redirect
  const country = request.geo?.country || 'US'
  if (pathname === '/' && country === 'DE') {
    return NextResponse.redirect(new URL('/de', request.url))
  }

  return response
}

export const config = {
  matcher: [
    '/((?!_next/static|_next/image|favicon.ico).*)',
  ],
}
```

## Metadata and SEO

Configure metadata for improved SEO and social sharing.

```tsx
// app/layout.tsx - Static metadata
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: {
    template: '%s | My Site',
    default: 'My Site',
  },
  description: 'Welcome to my website',
  openGraph: {
    title: 'My Site',
    description: 'Welcome to my website',
    url: 'https://mysite.com',
    siteName: 'My Site',
    images: ['/og-image.jpg'],
    locale: 'en_US',
    type: 'website',
  },
  twitter: {
    card: 'summary_large_image',
    title: 'My Site',
    description: 'Welcome to my website',
    images: ['/twitter-image.jpg'],
  },
  robots: {
    index: true,
    follow: true,
  },
}

// app/blog/[slug]/page.tsx - Dynamic metadata
export async function generateMetadata({
  params,
}: {
  params: Promise<{ slug: string }>
}): Promise<Metadata> {
  const { slug } = await params
  const post = await getPost(slug)

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.coverImage],
    },
  }
}
```

## Environment Variables

Access environment variables securely in your application.

```tsx
// Server-side only (not exposed to browser)
const apiKey = process.env.API_SECRET_KEY
const dbUrl = process.env.DATABASE_URL

// Client-side accessible (prefixed with NEXT_PUBLIC_)
const publicApiUrl = process.env.NEXT_PUBLIC_API_URL

// .env.local
// API_SECRET_KEY=secret123
// DATABASE_URL=postgres://...
// NEXT_PUBLIC_API_URL=https://api.example.com

// Usage in Server Component
export default async function Page() {
  const data = await fetch(process.env.API_URL!, {
    headers: {
      Authorization: `Bearer ${process.env.API_KEY}`,
    },
  })

  return <div>{/* ... */}</div>
}

// Usage in Client Component
'use client'

export function ApiStatus() {
  // Only NEXT_PUBLIC_ variables available
  const apiUrl = process.env.NEXT_PUBLIC_API_URL

  return <p>API: {apiUrl}</p>
}
```

## next.config.ts

Configure Next.js behavior with the configuration file.

```typescript
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  // Enable Cache Components (use cache directive)
  cacheComponents: true,

  // Enable React Compiler
  reactCompiler: true,

  // Turbopack configuration
  turbopack: {
    resolveAlias: {
      '@components': './src/components',
    },
  },

  // Image optimization
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '**.example.com',
      },
    ],
    formats: ['image/avif', 'image/webp'],
  },

  // Redirects
  async redirects() {
    return [
      {
        source: '/old-page',
        destination: '/new-page',
        permanent: true,
      },
    ]
  },

  // Rewrites
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: 'https://api.example.com/:path*',
      },
    ]
  },

  // Headers
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
        ],
      },
    ]
  },
}

export default nextConfig
```

---

Next.js is ideal for building production-ready web applications that require excellent performance, SEO optimization, and developer experience. Common use cases include e-commerce platforms, marketing websites, SaaS dashboards, blogs, documentation sites, and any application that benefits from server-side rendering or static generation. The framework excels at hybrid applications where some pages are statically generated while others require real-time data.

Integration patterns typically involve connecting Next.js with headless CMS platforms (Contentful, Sanity, Strapi), databases (PostgreSQL, MongoDB via Prisma), authentication providers (NextAuth.js, Clerk, Auth0), and deployment platforms (Vercel, AWS, Docker). The App Router's Server Components architecture enables efficient data fetching patterns where database queries can be made directly in components without exposing credentials to the client, while Server Actions provide a seamless way to handle form submissions and data mutations with built-in progressive enhancement.
