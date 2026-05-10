# Rendering Strategies & Performance Notes

---

# 1. Prerequisites

## 1.1 Hydration

Hydration is the **bridge between static server-rendered HTML and a fully interactive React app**.

### The Problem

SSR/SSG sends HTML that looks functional but behaves like a static photo.

- Buttons are visible
- UI is rendered
- But interactions do not work yet

This happens because the JavaScript "brains" have not loaded.

### The Solution

1. Browser downloads JavaScript
2. React walks through existing HTML
3. React attaches event listeners
4. UI becomes interactive

> The period between visible HTML and completed interactivity is called the **"Uncanny Valley"**

---

## 1.2 Pre-rendering

**Pre-rendering** is an umbrella term for any strategy that generates HTML before reaching the browser.

Includes:

- SSR
- SSG

---

## 1.3 Dynamic Rendering

Dynamic Rendering means switching between SSR and CSR based on the **User Agent**.

### Example

- Google Bot → SSR (better SEO)
- Human User → CSR

---

## 1.4 Edge Rendering

Edge Rendering moves SSR closer to users using **Edge Nodes**.

### Benefits

- Lower latency
- Faster SSR
- CDN-like speed with server execution capability

---

## 1.5 Render Blocking

Render-blocking resources prevent the browser from painting the page until they are fully processed.

### Common Render-Blocking Resources

- Scripts
- Stylesheets

### What Happens

Browser pauses:

- DOM construction
- CSSOM construction

Result:

- Blank white screen
- Delayed First Contentful Paint (FCP)

---

## 1.6 Critical Path CSS

Critical Path CSS extracts only the CSS needed for **Above-the-Fold** content (the part of the page a user sees immediately without scrolling).

### Goal

Render visible content immediately without waiting for the full stylesheet.

---

### How the Process Works

1. Identify Above-the-Fold content
2. Extract required CSS
3. Inline CSS inside:
   - `<style>` in `<head>`
4. Defer remaining CSS asynchronously

---

# 2. Client-Side Rendering (CSR)

In CSR, the server sends:

- Minimal HTML
- Large JavaScript bundle

The browser executes JS to render the application.

---

### How It Works

User sees:

- Blank screen OR
- Loading spinner

Until JavaScript finishes downloading and executing.

---

### Best For

- Dashboards
- SaaS products
- Authenticated apps
- Non-SEO applications

---

### Pros

- Fast client-side transitions
- Desktop-app-like feel

---

### Cons

- Poor SEO
- Slow initial paint on weak devices

---

# 3. Server-Side Rendering (SSR)

With SSR, the server generates full HTML on **every request**.

Browser receives a complete page immediately.

---

### How It Works

1. Server fetches data
2. Server renders HTML
3. HTML sent to browser
4. Hydration attaches interactivity

---

### Best For

- Personalized homepages
- Social feeds
- Frequently changing content

---

### Pros

- Excellent SEO
- Faster perceived loading

---

### Cons

- Higher server overhead
- Every navigation may require server round-trip

---

## 3.1 Server-Side Rendering with Data

Use:

- `getServerSideProps`

This async function runs on every request.

---

# 4. Static Site Generation (SSG)

SSG generates HTML at **build time**.

Static files are deployed directly to a CDN.

---

### How It Works

1. Fetch data during build
2. Generate static HTML
3. Serve via CDN

---

### Best For

- Blogs
- Documentation
- Marketing websites

---

### Pros

- Extremely fast
- Cheap hosting
- Secure

---

### Cons

- Large build times for huge sites
- Content becomes stale until rebuild

---

## 4.1 Static Generation with Data

### Use `getStaticProps`

When page content depends on external data.

### Use `getStaticPaths`

When dynamic routes depend on external data.

Usually combined with:

- `getStaticProps`

---

# 5. Incremental Static Regeneration (ISR)

ISR combines benefits of SSG and SSR.

Popularized by:

- `Next.js`

---

### How It Works

Pages stay static but refresh in the background.

Example:

> Revalidate every 60 seconds

---

### Best For

- E-commerce products
- Large news sites

---

### Pros

- Static-level speed
- Fresh content
- Avoids massive rebuilds

---

### Cons

- Temporary stale window during revalidation

---

## 5.1 Next.js ISR

Implemented using:

- `getStaticProps`
- `revalidate`

---

# 6. Performance Metrics (Core Web Vitals)

These metrics measure rendering performance.

---

## 6.1 TTFB (Time to First Byte)

Time browser waits for first byte from server.

---

## 6.2 FCP (First Contentful Paint)

Moment first visible content appears.

Examples:

- Logo
- Text
- Loading bar

---

## 6.3 LCP (Largest Contentful Paint)

When primary content becomes visible.

Examples:

- Hero image
- Main heading

This is the:

> "Is it loaded yet?" metric

---

## 6.4 TTI (Time to Interactive)

When hydration finishes and page responds to user interaction.

---

# 7. Rendering Strategy Comparison

| Metric | CSR                     | SSR                           | SSG                      | ISR                      |
| ------ | ----------------------- | ----------------------------- | ------------------------ | ------------------------ |
| TTFB   | Fastest (Static shell)  | Slower (Server computes HTML) | Fastest (CDN)            | Fastest (CDN)            |
| FCP    | Slow (Waiting for JS)   | Fast (Ready HTML)             | Fastest (Pre-built HTML) | Fastest (Pre-built HTML) |
| LCP    | Slow (Waiting for Data) | Fast (Complete HTML)          | Fastest                  | Fastest                  |
| TTI    | Slow (Heavy JS load)    | Moderate (Hydration required) | Moderate                 | Moderate                 |
| SEO    | Low                     | High                          | High                     | High                     |

---

# 8. Key Takeaways

## 8.1 Server Delay (TTFB)

SSR has highest TTFB because server generates HTML dynamically.

SSG/ISR are fastest because files already exist.

---

## 8.2 Blank Screen Problem (FCP)

CSR suffers most because browser waits for JavaScript execution.

---

## 8.3 The "Uncanny Valley" (TTI)

SSR, SSG, and ISR all experience temporary non-interactive UI during hydration.

User may:

- Click buttons
- Open menus

Before JS becomes active.

---

## 8.4 ISR Consistency

ISR provides SSG-like performance while refreshing stale content in the background.

---

# 9. Golden Rules of Performance

- Fastest initial visit:
  - **SSG / ISR**

- Smoothest transitions after initial load:
  - **CSR**
  - Hybrid frameworks like `Next.js`

- User-specific dynamic data:
  - **SSR or CSR**
