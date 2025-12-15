# Core Web Vitals

## Overview

Core Web Vitals are a set of standardized metrics from Google that measure real-world user experience on the web. They focus on three critical aspects: loading performance, interactivity, and visual stability. These metrics are crucial for SEO rankings and user satisfaction.

## Table of Contents
- [What are Core Web Vitals](#what-are-core-web-vitals)
- [LCP - Largest Contentful Paint](#lcp---largest-contentful-paint)
- [INP/FID - Interaction to Next Paint](#inp---interaction-to-next-paint)
- [CLS - Cumulative Layout Shift](#cls---cumulative-layout-shift)
- [Measuring Core Web Vitals](#measuring-core-web-vitals)
- [Optimization Strategies](#optimization-strategies)
- [Interview Questions](#interview-questions)

## What are Core Web Vitals

### The Three Pillars

```javascript
// Core Web Vitals Metrics
const coreWebVitals = {
  LCP: 'Largest Contentful Paint',      // Loading Performance
  INP: 'Interaction to Next Paint',      // Interactivity (replaced FID)
  CLS: 'Cumulative Layout Shift'         // Visual Stability
};

// Target Thresholds
const thresholds = {
  LCP: { good: 2500, poor: 4000 },       // milliseconds
  INP: { good: 200, poor: 500 },         // milliseconds
  CLS: { good: 0.1, poor: 0.25 }         // score
};
```

### Why They Matter

**Business Impact:**
- 1 second delay in page load = 7% reduction in conversions
- 0.1 second improvement in LCP = 8% increase in conversion rate
- Poor Core Web Vitals = Lower Google search rankings

**User Experience:**
- Faster loading = Better engagement
- Stable layouts = Less frustration
- Responsive interactions = Higher satisfaction

## LCP - Largest Contentful Paint

### What is LCP?

LCP measures loading performance by tracking when the **largest content element** becomes visible in the viewport.

```javascript
// Good LCP: < 2.5s
// Needs Improvement: 2.5s - 4s
// Poor LCP: > 4s

// Elements that can be LCP:
// - <img> elements
// - <image> inside <svg>
// - <video> elements (poster image)
// - Background images with url()
// - Block-level elements with text
```

### Common LCP Elements

```html
<!-- Example 1: Hero Image (Most Common) -->
<section class="hero">
  <img src="hero-image.jpg" alt="Hero" />
  <!-- This image is often the LCP element -->
</section>

<!-- Example 2: Text Block -->
<article>
  <h1>Main Headline</h1>
  <p>Large paragraph...</p>
  <!-- H1 or first paragraph might be LCP -->
</article>

<!-- Example 3: Background Image -->
<div style="background-image: url('banner.jpg'); height: 500px;">
  <!-- Background image can be LCP -->
</div>
```

### Identifying LCP Element

```javascript
// Method 1: PerformanceObserver
const observer = new PerformanceObserver((list) => {
  const entries = list.getEntries();
  const lastEntry = entries[entries.length - 1];

  console.log('LCP Element:', lastEntry.element);
  console.log('LCP Time:', lastEntry.renderTime || lastEntry.loadTime);
});

observer.observe({ type: 'largest-contentful-paint', buffered: true });

// Method 2: Web Vitals Library
import { onLCP } from 'web-vitals';

onLCP((metric) => {
  console.log('LCP:', metric.value);
  console.log('Element:', metric.entries[metric.entries.length - 1].element);
});
```

### Optimizing LCP

**1. Optimize Images**
```html
<!-- Bad: Large unoptimized image -->
<img src="hero.jpg" alt="Hero" />

<!-- Good: Modern format, responsive, priority -->
<img
  src="hero.avif"
  srcset="hero-400.avif 400w, hero-800.avif 800w, hero-1200.avif 1200w"
  sizes="(max-width: 600px) 400px, (max-width: 1200px) 800px, 1200px"
  alt="Hero"
  loading="eager"
  fetchpriority="high"
/>
```

**2. Preload Critical Resources**
```html
<!-- Preload LCP image -->
<link rel="preload" as="image" href="hero.avif" fetchpriority="high" />

<!-- Preload critical fonts -->
<link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin />

<!-- Preconnect to CDN -->
<link rel="preconnect" href="https://cdn.example.com" />
```

**3. Optimize Server Response Time**
```javascript
// Use CDN for static assets
const imageUrl = 'https://cdn.example.com/optimized/hero.avif';

// Implement caching
// Cache-Control: public, max-age=31536000, immutable

// Use HTTP/2 Server Push
// Link: </critical.css>; rel=preload; as=style
```

**4. Remove Render-Blocking Resources**
```html
<!-- Bad: Blocking CSS -->
<link rel="stylesheet" href="styles.css" />

<!-- Good: Critical CSS inline, async rest -->
<style>
  /* Critical above-the-fold CSS */
  .hero { ... }
</style>
<link rel="preload" href="styles.css" as="style"
      onload="this.onload=null;this.rel='stylesheet'" />
<noscript><link rel="stylesheet" href="styles.css" /></noscript>
```

**5. React/Next.js Optimization**
```jsx
// Next.js Image Component (automatic optimization)
import Image from 'next/image';

function Hero() {
  return (
    <Image
      src="/hero.jpg"
      alt="Hero"
      width={1200}
      height={600}
      priority  // Preload LCP image
      quality={90}
    />
  );
}

// React lazy loading for non-critical components
import { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <>
      {/* Critical content loads immediately */}
      <Hero />

      {/* Non-critical content lazy loads */}
      <Suspense fallback={<div>Loading...</div>}>
        <HeavyComponent />
      </Suspense>
    </>
  );
}
```

## INP - Interaction to Next Paint

### What is INP?

INP measures **responsiveness** by tracking the latency of all user interactions (clicks, taps, keyboard) throughout the page lifecycle. It replaced FID (First Input Delay) in 2024.

```javascript
// Good INP: < 200ms
// Needs Improvement: 200ms - 500ms
// Poor INP: > 500ms

// INP measures:
// - Input delay (time to process event)
// - Processing time (event handler execution)
// - Presentation delay (rendering updates)
```

### INP vs FID

```javascript
// FID (First Input Delay) - OLD
// - Only measured FIRST interaction
// - Only measured input delay (not processing or rendering)

// INP (Interaction to Next Paint) - NEW
// - Measures ALL interactions
// - Includes input delay + processing + rendering
// - More comprehensive responsiveness metric
```

### Measuring INP

```javascript
// Using Web Vitals Library
import { onINP } from 'web-vitals';

onINP((metric) => {
  console.log('INP:', metric.value);
  console.log('Rating:', metric.rating); // 'good', 'needs-improvement', 'poor'

  // Send to analytics
  sendToAnalytics({
    metric: 'INP',
    value: metric.value,
    rating: metric.rating
  });
});

// Manual PerformanceObserver
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.interactionId > 0) {
      const duration = entry.processingEnd - entry.processingStart;
      console.log('Interaction duration:', duration);
    }
  }
});

observer.observe({ type: 'event', buffered: true, durationThreshold: 0 });
```

### Optimizing INP

**1. Reduce JavaScript Execution Time**
```javascript
// Bad: Long synchronous task
function processLargeDataset(data) {
  for (let i = 0; i < data.length; i++) {
    // Heavy processing blocks main thread
    expensiveOperation(data[i]);
  }
}

// Good: Break into chunks with setTimeout
function processLargeDataset(data) {
  const chunkSize = 100;
  let index = 0;

  function processChunk() {
    const end = Math.min(index + chunkSize, data.length);

    for (let i = index; i < end; i++) {
      expensiveOperation(data[i]);
    }

    index = end;

    if (index < data.length) {
      setTimeout(processChunk, 0); // Yield to browser
    }
  }

  processChunk();
}

// Better: Use scheduler.yield() (modern browsers)
async function processLargeDataset(data) {
  for (let i = 0; i < data.length; i++) {
    expensiveOperation(data[i]);

    if (i % 100 === 0) {
      await scheduler.yield(); // Yield to browser
    }
  }
}
```

**2. Debounce and Throttle**
```javascript
// Debounce: Execute after user stops typing
function debounce(func, wait) {
  let timeout;
  return function executedFunction(...args) {
    clearTimeout(timeout);
    timeout = setTimeout(() => func.apply(this, args), wait);
  };
}

// Usage: Search input
const searchHandler = debounce((query) => {
  fetchSearchResults(query);
}, 300);

input.addEventListener('input', (e) => searchHandler(e.target.value));

// Throttle: Execute at most once per interval
function throttle(func, limit) {
  let inThrottle;
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

// Usage: Scroll handler
const scrollHandler = throttle(() => {
  updateScrollPosition();
}, 100);

window.addEventListener('scroll', scrollHandler);
```

**3. Use Web Workers for Heavy Computation**
```javascript
// worker.js
self.addEventListener('message', (e) => {
  const result = heavyComputation(e.data);
  self.postMessage(result);
});

function heavyComputation(data) {
  // CPU-intensive work
  return processedData;
}

// main.js
const worker = new Worker('worker.js');

button.addEventListener('click', () => {
  // Offload to worker (doesn't block main thread)
  worker.postMessage(largeDataset);
});

worker.addEventListener('message', (e) => {
  displayResults(e.data);
});
```

**4. Optimize React Event Handlers**
```jsx
// Bad: Creating new function on every render
function App() {
  return (
    <button onClick={() => handleClick()}>
      Click me
    </button>
  );
}

// Good: Use useCallback
import { useCallback } from 'react';

function App() {
  const handleClick = useCallback(() => {
    // Handle click
  }, []);

  return <button onClick={handleClick}>Click me</button>;
}

// Good: Optimize expensive operations
import { useMemo, useState } from 'react';

function DataTable({ data }) {
  const [sortColumn, setSortColumn] = useState('name');

  // Expensive sorting only recalculates when dependencies change
  const sortedData = useMemo(() => {
    return [...data].sort((a, b) => {
      return a[sortColumn] > b[sortColumn] ? 1 : -1;
    });
  }, [data, sortColumn]);

  return <Table data={sortedData} />;
}
```

## CLS - Cumulative Layout Shift

### What is CLS?

CLS measures **visual stability** by tracking unexpected layout shifts that occur during the page's lifetime.

```javascript
// Good CLS: < 0.1
// Needs Improvement: 0.1 - 0.25
// Poor CLS: > 0.25

// CLS Score Calculation:
// layout shift score = impact fraction × distance fraction

// Impact fraction: % of viewport affected
// Distance fraction: Distance element moved / viewport height
```

### Common Causes of CLS

**1. Images/Videos Without Dimensions**
```html
<!-- Bad: No dimensions -->
<img src="image.jpg" alt="Image" />
<!-- Browser doesn't know size until loaded → layout shift -->

<!-- Good: Specify dimensions -->
<img src="image.jpg" alt="Image" width="800" height="600" />

<!-- Good: Use aspect ratio -->
<img
  src="image.jpg"
  alt="Image"
  style="aspect-ratio: 16/9; width: 100%;"
/>
```

**2. Dynamically Injected Content**
```html
<!-- Bad: Ad loads and pushes content down -->
<article>
  <h1>Article Title</h1>
  <div id="ad-slot"></div> <!-- Ad loads later, causes shift -->
  <p>Article content...</p>
</article>

<!-- Good: Reserve space -->
<article>
  <h1>Article Title</h1>
  <div id="ad-slot" style="min-height: 250px;">
    <!-- Ad loads into reserved space -->
  </div>
  <p>Article content...</p>
</article>
```

**3. Web Fonts Loading**
```css
/* Bad: FOUT (Flash of Unstyled Text) causes shift */
@font-face {
  font-family: 'CustomFont';
  src: url('font.woff2');
}

body {
  font-family: 'CustomFont', sans-serif;
}

/* Good: Use font-display to control behavior */
@font-face {
  font-family: 'CustomFont';
  src: url('font.woff2');
  font-display: swap; /* or optional */
}

/* Better: Preload critical fonts */
/* <link rel="preload" href="font.woff2" as="font" crossorigin /> */
```

### Measuring CLS

```javascript
// Using Web Vitals Library
import { onCLS } from 'web-vitals';

onCLS((metric) => {
  console.log('CLS:', metric.value);
  console.log('Entries:', metric.entries);

  // Log elements causing shifts
  metric.entries.forEach((entry) => {
    console.log('Shifted element:', entry.sources);
  });
});

// Manual PerformanceObserver
let clsScore = 0;

const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (!entry.hadRecentInput) {
      clsScore += entry.value;
      console.log('Layout shift:', entry.value);
      console.log('Elements:', entry.sources);
    }
  }
});

observer.observe({ type: 'layout-shift', buffered: true });
```

### Optimizing CLS

**1. Reserve Space for Dynamic Content**
```jsx
// React skeleton loading
function ProductCard({ loading, product }) {
  if (loading) {
    return (
      <div className="skeleton">
        <div className="skeleton-image" style={{ height: '200px' }} />
        <div className="skeleton-title" style={{ height: '24px' }} />
        <div className="skeleton-price" style={{ height: '20px' }} />
      </div>
    );
  }

  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>{product.price}</p>
    </div>
  );
}
```

**2. CSS Aspect Ratio**
```css
/* Modern aspect ratio boxes */
.image-container {
  aspect-ratio: 16 / 9;
  width: 100%;
}

.image-container img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* Fallback for older browsers */
.image-container {
  position: relative;
  padding-bottom: 56.25%; /* 16:9 ratio */
}

.image-container img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
```

**3. Avoid Inserting Content Above Existing Content**
```javascript
// Bad: Prepend notification (pushes content down)
function showNotification(message) {
  const notification = createNotificationElement(message);
  document.body.prepend(notification); // CLS!
}

// Good: Fixed/absolute positioning
function showNotification(message) {
  const notification = createNotificationElement(message);
  notification.style.position = 'fixed';
  notification.style.top = '20px';
  notification.style.right = '20px';
  document.body.append(notification); // No CLS!
}
```

**4. Optimize Font Loading**
```html
<!-- Preload critical fonts -->
<link
  rel="preload"
  href="/fonts/main-font.woff2"
  as="font"
  type="font/woff2"
  crossorigin
/>

<style>
  @font-face {
    font-family: 'MainFont';
    src: url('/fonts/main-font.woff2') format('woff2');
    font-display: swap; /* Fallback font while loading */
  }

  /* Size adjust to match fallback font */
  @font-face {
    font-family: 'MainFont';
    size-adjust: 105%; /* Adjust to minimize shift */
  }
</style>
```

## Measuring Core Web Vitals

### Using Web Vitals Library

```bash
npm install web-vitals
```

```javascript
// Complete implementation
import { onCLS, onINP, onLCP } from 'web-vitals';

function sendToAnalytics({ name, value, rating, delta, id }) {
  // Send to Google Analytics
  gtag('event', name, {
    event_category: 'Web Vitals',
    value: Math.round(name === 'CLS' ? delta * 1000 : delta),
    event_label: id,
    non_interaction: true,
  });

  // Or send to custom endpoint
  fetch('/api/vitals', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ name, value, rating, id })
  });
}

onCLS(sendToAnalytics);
onINP(sendToAnalytics);
onLCP(sendToAnalytics);
```

### Using Chrome DevTools

```javascript
// 1. Open DevTools → Performance tab
// 2. Click Record
// 3. Interact with page
// 4. Stop recording
// 5. Look for:
//    - LCP in "Timings" lane
//    - Long tasks in "Main" lane
//    - Layout shifts in "Experience" lane

// 6. Use Lighthouse
// 7. Generate report for real metrics
```

### Field Data vs Lab Data

```javascript
// Field Data (Real User Monitoring)
// - Actual user experiences
// - Collected via web-vitals library
// - Available in Chrome User Experience Report
// - What Google uses for ranking

// Lab Data (Synthetic Testing)
// - Lighthouse
// - WebPageTest
// - Controlled environment
// - Useful for debugging
// - May differ from field data
```

## Optimization Strategies

### Priority Matrix

```javascript
const optimizationPriority = {
  high: [
    'Optimize LCP element (hero image)',
    'Remove render-blocking resources',
    'Implement image lazy loading',
    'Add image dimensions',
    'Preload critical fonts'
  ],
  medium: [
    'Code splitting',
    'Debounce/throttle event handlers',
    'Optimize JavaScript execution',
    'Implement service worker caching'
  ],
  low: [
    'Fine-tune bundle size',
    'Optimize third-party scripts',
    'Advanced image formats (AVIF)'
  ]
};
```

### Complete Example: Optimized Landing Page

```jsx
// app/page.jsx (Next.js 13+)
import Image from 'next/image';
import { Suspense, lazy } from 'react';

// Lazy load below-the-fold components
const Reviews = lazy(() => import('./Reviews'));
const Newsletter = lazy(() => import('./Newsletter'));

export default function LandingPage() {
  return (
    <>
      {/* Critical above-the-fold content */}
      <Hero />

      {/* Non-critical content with lazy loading */}
      <Suspense fallback={<div style={{ height: '400px' }} />}>
        <Reviews />
      </Suspense>

      <Suspense fallback={<div style={{ height: '200px' }} />}>
        <Newsletter />
      </Suspense>
    </>
  );
}

function Hero() {
  return (
    <section className="hero">
      {/* Optimized LCP element */}
      <Image
        src="/hero.avif"
        alt="Hero Image"
        width={1200}
        height={600}
        priority  // Preload
        quality={90}
        placeholder="blur"
        blurDataURL="data:image/jpeg;base64,..."
      />

      <h1>Welcome to Our Site</h1>
      <button onClick={handleClick}>Get Started</button>
    </section>
  );
}

// Optimized event handler
import { useCallback } from 'react';

function handleClick() {
  const optimizedHandler = useCallback(() => {
    // Handle click
  }, []);

  return optimizedHandler;
}
```

```html
<!-- index.html head -->
<head>
  <!-- Preconnect to critical origins -->
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://cdn.example.com" />

  <!-- Preload critical resources -->
  <link rel="preload" href="/hero.avif" as="image" fetchpriority="high" />
  <link rel="preload" href="/font.woff2" as="font" type="font/woff2" crossorigin />

  <!-- Critical CSS inline -->
  <style>
    .hero { min-height: 600px; }
    /* ... critical styles ... */
  </style>

  <!-- Async non-critical CSS -->
  <link rel="preload" href="/styles.css" as="style"
        onload="this.onload=null;this.rel='stylesheet'" />
</head>
```

## Interview Questions

**Q1: What are Core Web Vitals and why do they matter?**
A: Core Web Vitals are Google's standardized metrics for user experience:
- **LCP** (Loading): Measures when main content loads (< 2.5s)
- **INP** (Interactivity): Measures responsiveness to user input (< 200ms)
- **CLS** (Visual Stability): Measures unexpected layout shifts (< 0.1)

**Impact**: Directly affect Google rankings, user engagement, and conversion rates. Pages with good CWV see better SEO and higher user satisfaction.

**Q2: How would you optimize LCP for a hero image?**
A:
```jsx
// 1. Use Next.js Image with priority
<Image src="/hero.avif" priority quality={90} />

// 2. Preload in HTML
<link rel="preload" as="image" href="/hero.avif" />

// 3. Use modern formats (AVIF/WebP)
// 4. Optimize file size (compression)
// 5. Use CDN for fast delivery
// 6. Implement responsive images
<img srcset="hero-400.avif 400w, hero-800.avif 800w" />
```

**Q3: Difference between FID and INP?**
A:
- **FID** (Old): Measured ONLY first interaction input delay
- **INP** (New): Measures ALL interactions including processing and rendering
- INP is more comprehensive and better represents overall responsiveness
- INP replaced FID as a Core Web Vital in March 2024

**Q4: What causes CLS and how do you fix it?**
A: Common causes:
1. **Images without dimensions** → Add width/height attributes
2. **Web fonts loading** → Use font-display: swap, preload fonts
3. **Dynamic content injection** → Reserve space with min-height
4. **Ads** → Set container dimensions before loading

```css
/* Fix with aspect ratio */
img { aspect-ratio: 16/9; width: 100%; }
```

**Q5: How do you measure Core Web Vitals?**
A:
```javascript
// Production: web-vitals library
import { onCLS, onINP, onLCP } from 'web-vitals';

onLCP((metric) => sendToAnalytics(metric));

// Development: Chrome DevTools + Lighthouse
// Field data: Chrome User Experience Report
// Lab testing: Lighthouse, WebPageTest
```

**Q6: What's the difference between field and lab data?**
A:
- **Field Data**: Real user measurements, varies by device/network, what Google uses for ranking
- **Lab Data**: Controlled environment (Lighthouse), consistent but may not reflect real users
- Both are important: Lab for debugging, Field for real-world performance

**Q7: How would you debug a poor INP score?**
A:
```javascript
// 1. Use Performance Observer to find slow interactions
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.duration > 200) {
      console.log('Slow interaction:', entry);
    }
  }
});
observer.observe({ type: 'event', durationThreshold: 200 });

// 2. Solutions:
// - Debounce/throttle event handlers
// - Break long tasks into chunks
// - Use Web Workers for heavy computation
// - Optimize React re-renders with useMemo/useCallback
```

**Q8: What's the critical rendering path and how does it affect CWV?**
A: Sequence of steps browser takes to render page:
1. HTML → DOM
2. CSS → CSSOM
3. DOM + CSSOM → Render Tree
4. Layout → Paint

**Impact on CWV**:
- Blocking CSS/JS delays LCP
- Long JavaScript execution delays INP
- Unoptimized rendering causes CLS

**Optimize**: Minimize blocking resources, inline critical CSS, defer non-critical JS

**Q9: How do you optimize web fonts to prevent CLS?**
A:
```html
<!-- Preload critical fonts -->
<link rel="preload" href="font.woff2" as="font" crossorigin />

<style>
  @font-face {
    font-family: 'MyFont';
    src: url('font.woff2') format('woff2');
    font-display: swap; /* Show fallback immediately */
  }
</style>

<!-- Use CSS font-loading API -->
<script>
  document.fonts.ready.then(() => {
    document.body.classList.add('fonts-loaded');
  });
</script>
```

**Q10: What tools do you use for performance optimization?**
A:
- **Lighthouse**: Overall performance score
- **Chrome DevTools**: Performance profiling, network waterfall
- **Web Vitals Library**: Real user monitoring
- **WebPageTest**: Detailed waterfall analysis
- **Bundle Analyzer**: JavaScript bundle size
- **PageSpeed Insights**: Field + lab data

## Summary

**Core Web Vitals Quick Reference:**
- **LCP** < 2.5s: Optimize images, preload resources, reduce server time
- **INP** < 200ms: Debounce/throttle, break long tasks, use Web Workers
- **CLS** < 0.1: Set dimensions, reserve space, optimize fonts

**Optimization Priority:**
1. Fix LCP element (usually hero image)
2. Add image dimensions to prevent CLS
3. Debounce expensive event handlers for INP
4. Preload critical resources
5. Remove render-blocking resources

**Key Takeaways:**
- Core Web Vitals directly impact SEO rankings
- Measure both field (real users) and lab (synthetic) data
- Focus on the biggest impact items first
- Use web-vitals library for production monitoring
- Test on real devices and networks

---

[Next: Lazy Loading →](./02-lazy-loading.md) | [← Back to Web Performance](./README.md)
