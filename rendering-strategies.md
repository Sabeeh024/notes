## Prerequisites 

### Hydration

Hydration is the "bridge" between a static server-rendered page and a fully functional React app.

The Problem: SSR/SSG sends HTML that looks like a webpage but behaves like a photo—you can see the buttons, but clicking them does nothing because the "brains" (JavaScript) haven't arrived yet.

The Solution: The browser downloads the JS, React "walks" through the HTML, and attaches event listeners to the DOM.

Note: The time period between user seeing HTML and till the hydration is complete means UI is interactive is called "Uncanny Valley"

### Pre-rendering: 

An umbrella term for any strategy that generates HTML before it reaches the browser (includes both SSR and SSG).

### Dynamic Rendering: 

Switching between SSR and CSR based on the "User Agent." For example, serving SSR to Google’s bot (for SEO) but CSR to a regular human user.

### Edge Rendering: 

Moving the SSR process away from a central server and onto "Edge Nodes" (servers physically closer to the user, like a CDN with code-running capabilities). This makes SSR almost as fast as SSG.

### Render Blocking: 

Render-blocking resources are files—primarily scripts and stylesheets—that prevent a browser from rendering the visible content of a page until they are fully downloaded, parsed, and executed.

When a browser encounters a render-blocking resource, it pauses the construction of the DOM (Document Object Model) and the CSSOM (CSS Object Model). Since the browser cannot paint the page without these "blueprints," the user sees a blank white screen or a partially loaded page, leading to a poor First Contentful Paint (FCP).

### Critical Path CSS: 

Critical Path CSS is the technique of extracting the CSS required to render the "Above-the-Fold" content (the part of the page a user sees immediately without scrolling) and inlining it directly into the HTML <head>.

#### How the Process Works

1. Identify Above-the-Fold Content: Determine which elements are visible on a standard viewport (e.g., header, hero image, navigation, first paragraph).
2. Extract the CSS: Pull only the styles necessary for those specific elements.
3. Inline the Critical CSS: Place these styles inside <style> tags in the <head> of your HTML document.
4. Defer Remaining CSS: Load the rest of your "non-critical" styles asynchronously so they don't block the initial render.

## Client-Side Rendering (CSR)

In CSR, the server sends a bare-bones HTML file and a large JavaScript bundle. The browser then executes the JS to "paint" the entire application.

How it works: The user sees a blank screen or a loading spinner until the JavaScript is fully downloaded and executed.

Best for: Highly interactive dashboards, SaaS products, and applications behind a login where SEO isn't a priority.

Pros: Fast transitions after the initial load; feels like a desktop app.

Cons: Poor SEO; slow "Time to First Meaningful Paint" on slower devices.

## Server-Side Rendering (SSR)

With SSR, the server generates the full HTML for a page on every single request. The browser receives a complete document it can display immediately.

How it works: The server fetches data, populates the HTML, and sends it back. Once it hits the browser, "Hydration" occurs—the process where JavaScript attaches event listeners to the static HTML to make it interactive.

Best for: Content-heavy sites where data changes frequently (e.g., social media feeds, personalized homepages).

Pros: Excellent SEO; faster perceived load time.

Cons: High server overhead; every page transition requires a round-trip to the server.

### Server-Side Rendering with data

To use Server-side Rendering for a page, you need to export an async function called `getServerSideProps`. This function will be called by the server on every request.

## Static Site Generation (SSG)

SSG builds the entire website into static HTML files at build time (when you run the deploy command), rather than on every request.

How it works: All data is fetched upfront. The result is a folder of static files that can be served via a CDN (Content Delivery Network).

Best for: Blogs, documentation, and marketing sites where content doesn't change based on who is looking at it.

Pros: Blazing fast; incredibly cheap to host; highly secure.

Cons: Build times can become massive for sites with thousands of pages; content can become "stale" until the next rebuild.

### Static Generation with data

Some pages require fetching external data for prerendering. There are two scenarios, and one or both might apply. In each case, you can use these functions that `Next.js` provides:

1. Your page content depends on external data: Use `getStaticProps`.
2. Your page paths depend on external data: Use `getStaticPaths` (usually in addition to `getStaticProps`).

## Incremental Static Regeneration (ISR)

ISR is the "best of both worlds" evolution of SSG, popularized by frameworks like Next.js.

How it works: It allows you to update static pages in the background after the site has been deployed. You can tell the server: "Keep this page static, but check if there's new data every 60 seconds."

Best for: E-commerce product pages or large news sites.

Pros: Static speeds with dynamic freshness; scales to millions of pages without long build times.

Cons: Perpetual "stale" window where users see outdated data while the server updates the cache in the background.

can be acheived via `getStaticProps` with `revalidate` in next.js

## Performance Metrics (The "Core Web Vitals")

When testing rendering strategies, these are the three metrics that actually matter:

TTFB (Time to First Byte): How long the browser waits for the server to send the very first piece of data. 

FCP (First Contentful Paint): When the user sees the first piece of content (like a logo or loading bar).

LCP (Largest Contentful Paint): When the main content (the big image or text block) is visible. This is the "is it loaded yet?" moment.

TTI (Time to Interactive): The moment hydration is finished and the page actually responds to clicks.

| Metric                 | CSR                     | SSR                             | SSG                        | ISR                        |
| ---------------------- | ----------------------- | ------------------------------- | -------------------------- | -------------------------- |
| TTFB (Server Response) | Fastest (Static shell)  | Slower (Server must build page) | Fastest (Served from CDN)  | Fastest (Served from CDN)  |
| FCP (First Content)    | Slow (Waiting for JS)   | Fast (HTML arrives ready)       | Fastest (Pre-built HTML)   | Fastest (Pre-built HTML)   |
| LCP (Main Content)     | Slow (Waiting for Data) | Fast (HTML is complete)         | Fastest (Pre-built)        | Fastest (Pre-built)        |
| TTI (Interactive)      | Slower (Huge JS load)   | Moderate (Needs Hydration)      | Moderate (Needs Hydration) | Moderate (Needs Hydration) |
| SEO Score              | Low                     | High                            | High                       | High                       |

### Key Takeaways for your notes:

1. The "Server Delay" (TTFB)
SSR has the highest TTFB because the server is "computing" the HTML while the user waits. SSG/ISR have the lowest because the file already exists; the server just hands it over.

2. The "Blank Screen" Problem (FCP)
CSR suffers here. The browser gets the HTML immediately, but it's empty. The user sees nothing until the JavaScript bundle is downloaded and executed.

3. The "Uncanny Valley" (TTI)
SSR, SSG, and ISR all share a similar TTI challenge. Because the HTML is visible quickly, the user might try to click a menu button before the JavaScript has finished Hydrating. This creates a "frozen" feeling for a few milliseconds.

4. The Consistency of ISR
ISR matches SSG performance exactly for the end-user. The only difference is that the "background" update process ensures the SSG files don't stay stale forever.

### The "Golden Rule" of Performance:

- If you want the fastest possible initial visit: Use SSG or ISR.
- If you want the smoothest transitions after the first load: Use CSR (or a Hybrid like Next.js).
- If your data is unique to every user: You must use SSR or CSR.