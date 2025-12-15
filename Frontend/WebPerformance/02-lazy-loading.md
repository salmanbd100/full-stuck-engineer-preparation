# Lazy Loading

## Overview

Lazy loading is a performance optimization technique that defers loading of non-critical resources until they're actually needed. This reduces initial page load time, saves bandwidth, and improves Core Web Vitals.

## Table of Contents
- [Image Lazy Loading](#image-lazy-loading)
- [Component Lazy Loading](#component-lazy-loading)
- [Route-Based Code Splitting](#route-based-code-splitting)
- [Intersection Observer API](#intersection-observer-api)
- [Best Practices](#best-practices)
- [Interview Questions](#interview-questions)

## Image Lazy Loading

### Native Browser Lazy Loading

```html
<!-- Modern browsers support loading="lazy" -->
<img
  src="image.jpg"
  alt="Description"
  loading="lazy"
  width="800"
  height="600"
/>

<!-- Eager loading for above-the-fold images -->
<img
  src="hero.jpg"
  alt="Hero"
  loading="eager"
  width="1200"
  height="600"
/>

<!-- Auto (default): Browser decides -->
<img src="image.jpg" alt="Description" loading="auto" />
```

**Browser Support:**
```javascript
// Check if browser supports native lazy loading
if ('loading' in HTMLImageElement.prototype) {
  // Native lazy loading supported
  console.log('Native lazy loading available');
} else {
  // Fallback to Intersection Observer
  console.log('Need polyfill or fallback');
}
```

### Custom Lazy Loading with Intersection Observer

```javascript
// Lazy load images using Intersection Observer
class LazyImageLoader {
  constructor(options = {}) {
    this.options = {
      root: null,
      rootMargin: '50px',
      threshold: 0.01,
      ...options
    };

    this.observer = new IntersectionObserver(
      this.handleIntersection.bind(this),
      this.options
    );
  }

  handleIntersection(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const img = entry.target;
        this.loadImage(img);
        this.observer.unobserve(img);
      }
    });
  }

  loadImage(img) {
    const src = img.dataset.src;
    const srcset = img.dataset.srcset;

    // Create new image to preload
    const imageLoader = new Image();

    imageLoader.onload = () => {
      img.src = src;
      if (srcset) {
        img.srcset = srcset;
      }
      img.classList.add('loaded');
    };

    imageLoader.onerror = () => {
      img.classList.add('error');
    };

    imageLoader.src = src;
  }

  observe(images) {
    images.forEach(img => this.observer.observe(img));
  }
}

// Usage
const lazyLoader = new LazyImageLoader({
  rootMargin: '100px' // Load 100px before entering viewport
});

const images = document.querySelectorAll('img[data-src]');
lazyLoader.observe(images);
```

```html
<!-- HTML structure for custom lazy loading -->
<img
  data-src="image.jpg"
  data-srcset="image-400.jpg 400w, image-800.jpg 800w"
  alt="Description"
  class="lazy-image"
  style="background: #f0f0f0;"
/>

<style>
  .lazy-image {
    opacity: 0;
    transition: opacity 0.3s;
  }

  .lazy-image.loaded {
    opacity: 1;
  }
</style>
```

### Progressive Image Loading (Blur-up)

```jsx
// React component with blur-up effect
import { useState, useEffect } from 'react';

function ProgressiveImage({ placeholder, src, alt }) {
  const [imageSrc, setImageSrc] = useState(placeholder);
  const [imageLoaded, setImageLoaded] = useState(false);

  useEffect(() => {
    const img = new Image();
    img.src = src;
    img.onload = () => {
      setImageSrc(src);
      setImageLoaded(true);
    };
  }, [src]);

  return (
    <div className="progressive-image">
      <img
        src={imageSrc}
        alt={alt}
        className={imageLoaded ? 'loaded' : 'loading'}
      />
    </div>
  );
}

// Usage
<ProgressiveImage
  placeholder="data:image/jpeg;base64,/9j/4AAQSkZJRg..." // Tiny base64 image
  src="/high-res-image.jpg"
  alt="Description"
/>
```

```css
.progressive-image {
  position: relative;
  overflow: hidden;
}

.progressive-image img {
  width: 100%;
  height: auto;
  transition: filter 0.3s;
}

.progressive-image img.loading {
  filter: blur(10px);
  transform: scale(1.1);
}

.progressive-image img.loaded {
  filter: blur(0);
  transform: scale(1);
}
```

### Next.js Image Lazy Loading

```jsx
import Image from 'next/image';

function Gallery({ images }) {
  return (
    <div className="gallery">
      {images.map((image, index) => (
        <Image
          key={image.id}
          src={image.url}
          alt={image.alt}
          width={400}
          height={300}
          loading={index < 3 ? 'eager' : 'lazy'}  // First 3 eager, rest lazy
          placeholder="blur"
          blurDataURL={image.placeholder}
          quality={85}
        />
      ))}
    </div>
  );
}
```

## Component Lazy Loading

### React Lazy and Suspense

```jsx
import { lazy, Suspense } from 'react';

// Lazy load component
const HeavyComponent = lazy(() => import('./HeavyComponent'));
const UserDashboard = lazy(() => import('./UserDashboard'));
const AdminPanel = lazy(() => import('./AdminPanel'));

function App() {
  return (
    <div>
      {/* Critical content loads immediately */}
      <Header />
      <Hero />

      {/* Heavy component lazy loads */}
      <Suspense fallback={<LoadingSpinner />}>
        <HeavyComponent />
      </Suspense>

      {/* Conditional lazy loading */}
      {isAdmin ? (
        <Suspense fallback={<div>Loading admin panel...</div>}>
          <AdminPanel />
        </Suspense>
      ) : (
        <Suspense fallback={<div>Loading dashboard...</div>}>
          <UserDashboard />
        </Suspense>
      )}
    </div>
  );
}
```

### Named Exports Lazy Loading

```jsx
// Component with named exports
// MyComponent.jsx
export function MyComponent() {
  return <div>Main Component</div>;
}

export function AnotherComponent() {
  return <div>Another</div>;
}

// Lazy loading named exports
const MyComponent = lazy(() =>
  import('./MyComponent').then(module => ({
    default: module.MyComponent
  }))
);

const AnotherComponent = lazy(() =>
  import('./MyComponent').then(module => ({
    default: module.AnotherComponent
  }))
);
```

### Error Boundaries for Lazy Components

```jsx
import { Component, lazy, Suspense } from 'react';

// Error boundary component
class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Lazy loading error:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <div>Failed to load component. <button onClick={() => window.location.reload()}>Retry</button></div>;
    }

    return this.props.children;
  }
}

// Usage
const LazyComponent = lazy(() => import('./LazyComponent'));

function App() {
  return (
    <ErrorBoundary>
      <Suspense fallback={<Loading />}>
        <LazyComponent />
      </Suspense>
    </ErrorBoundary>
  );
}
```

### Preloading Components

```jsx
import { lazy, Suspense, useEffect } from 'react';

// Lazy load with preload
const AdminPanel = lazy(() => import('./AdminPanel'));

// Preload function
const preloadAdminPanel = () => {
  import('./AdminPanel');
};

function Navigation() {
  return (
    <nav>
      <Link
        to="/admin"
        onMouseEnter={preloadAdminPanel}  // Preload on hover
      >
        Admin Panel
      </Link>
    </nav>
  );
}

// Or preload based on route prefetch
function useRoutePreload(route) {
  useEffect(() => {
    const timer = setTimeout(() => {
      if (route === '/admin') {
        preloadAdminPanel();
      }
    }, 2000); // Preload after 2 seconds

    return () => clearTimeout(timer);
  }, [route]);
}
```

## Route-Based Code Splitting

### React Router Lazy Loading

```jsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Lazy load route components
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Products = lazy(() => import('./pages/Products'));
const ProductDetail = lazy(() => import('./pages/ProductDetail'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/products" element={<Products />} />
          <Route path="/products/:id" element={<ProductDetail />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

### Next.js Dynamic Imports

```jsx
// pages/index.jsx
import dynamic from 'next/dynamic';

// Client-side only component (no SSR)
const DynamicComponent = dynamic(() => import('../components/Heavy'), {
  ssr: false,
  loading: () => <p>Loading...</p>
});

// With named export
const DynamicComponentWithNamed = dynamic(
  () => import('../components/MyComponent').then(mod => mod.MyComponent),
  { loading: () => <Loading /> }
);

// Conditional loading
export default function Home() {
  const [showHeavy, setShowHeavy] = useState(false);

  return (
    <div>
      <button onClick={() => setShowHeavy(true)}>
        Load Heavy Component
      </button>
      {showHeavy && <DynamicComponent />}
    </div>
  );
}
```

### Vue.js Lazy Loading

```javascript
// Vue Router lazy loading
import { createRouter, createWebHistory } from 'vue-router';

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/',
      component: () => import('./views/Home.vue')
    },
    {
      path: '/about',
      component: () => import('./views/About.vue')
    },
    {
      path: '/dashboard',
      component: () => import('./views/Dashboard.vue')
    }
  ]
});

// Component lazy loading
export default {
  components: {
    HeavyComponent: () => import('./components/HeavyComponent.vue')
  }
};
```

## Intersection Observer API

### Basic Intersection Observer

```javascript
// Create observer
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      console.log('Element is visible:', entry.target);
      // Load content
      loadContent(entry.target);
      // Stop observing after loaded
      observer.unobserve(entry.target);
    }
  });
}, {
  root: null,                // viewport
  rootMargin: '0px',         // margin around root
  threshold: 0.1             // 10% visible triggers callback
});

// Observe elements
const elements = document.querySelectorAll('.lazy-load');
elements.forEach(el => observer.observe(el));
```

### Advanced Configuration

```javascript
const options = {
  // Element to use as viewport (null = browser viewport)
  root: document.querySelector('#scrollArea'),

  // Margin around root (can trigger earlier/later)
  rootMargin: '100px 0px',  // Load 100px before entering viewport

  // Threshold: 0-1, or array of thresholds
  threshold: [0, 0.25, 0.5, 0.75, 1]  // Multiple trigger points
};

const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    console.log('Intersection ratio:', entry.intersectionRatio);

    if (entry.intersectionRatio >= 0.5) {
      entry.target.classList.add('half-visible');
    }

    if (entry.isIntersecting) {
      entry.target.classList.add('fully-visible');
    }
  });
}, options);
```

### Infinite Scroll Implementation

```jsx
import { useEffect, useRef, useState } from 'react';

function InfiniteScroll() {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  const loaderRef = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        const first = entries[0];
        if (first.isIntersecting && !loading) {
          loadMore();
        }
      },
      { threshold: 1.0 }
    );

    const currentLoader = loaderRef.current;
    if (currentLoader) {
      observer.observe(currentLoader);
    }

    return () => {
      if (currentLoader) {
        observer.unobserve(currentLoader);
      }
    };
  }, [loading]);

  const loadMore = async () => {
    setLoading(true);
    const newItems = await fetchItems(page);
    setItems(prev => [...prev, ...newItems]);
    setPage(prev => prev + 1);
    setLoading(false);
  };

  return (
    <div>
      {items.map(item => (
        <ItemCard key={item.id} item={item} />
      ))}
      <div ref={loaderRef} className="loader">
        {loading && <Spinner />}
      </div>
    </div>
  );
}
```

### Lazy Load Sections

```jsx
function LazySection({ children, placeholder }) {
  const [isLoaded, setIsLoaded] = useState(false);
  const sectionRef = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsLoaded(true);
          observer.disconnect();
        }
      },
      { rootMargin: '200px' }
    );

    if (sectionRef.current) {
      observer.observe(sectionRef.current);
    }

    return () => observer.disconnect();
  }, []);

  return (
    <section ref={sectionRef}>
      {isLoaded ? children : placeholder}
    </section>
  );
}

// Usage
function App() {
  return (
    <>
      <Hero />

      <LazySection placeholder={<div style={{ height: '400px' }} />}>
        <Features />
      </LazySection>

      <LazySection placeholder={<div style={{ height: '600px' }} />}>
        <Testimonials />
      </LazySection>
    </>
  );
}
```

## Best Practices

### Loading Priority Strategy

```javascript
const loadingStrategy = {
  // Above the fold: Eager loading
  critical: {
    loading: 'eager',
    fetchpriority: 'high',
    preload: true
  },

  // Just below fold: Normal loading
  important: {
    loading: 'auto',
    fetchpriority: 'auto'
  },

  // Far below fold: Lazy loading
  deferred: {
    loading: 'lazy',
    rootMargin: '200px'  // Load before visible
  }
};
```

### Skeleton Loading

```jsx
function ProductList({ products, loading }) {
  if (loading) {
    return (
      <div className="product-grid">
        {[...Array(6)].map((_, i) => (
          <ProductSkeleton key={i} />
        ))}
      </div>
    );
  }

  return (
    <div className="product-grid">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

function ProductSkeleton() {
  return (
    <div className="skeleton-card">
      <div className="skeleton-image" />
      <div className="skeleton-title" />
      <div className="skeleton-price" />
    </div>
  );
}
```

```css
.skeleton-card {
  padding: 16px;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
}

.skeleton-image,
.skeleton-title,
.skeleton-price {
  background: linear-gradient(
    90deg,
    #f0f0f0 25%,
    #e0e0e0 50%,
    #f0f0f0 75%
  );
  background-size: 200% 100%;
  animation: loading 1.5s infinite;
}

.skeleton-image {
  height: 200px;
  margin-bottom: 12px;
}

.skeleton-title {
  height: 20px;
  margin-bottom: 8px;
}

.skeleton-price {
  height: 16px;
  width: 60%;
}

@keyframes loading {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
```

### Content Visibility CSS

```css
/* Modern CSS property for lazy rendering */
.lazy-content {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px; /* Reserve space */
}

/* Browser won't render until visible */
.below-fold-section {
  content-visibility: auto;
  contain-intrinsic-size: auto 800px;
}
```

## Interview Questions

**Q1: What is lazy loading and why use it?**
A: Lazy loading defers loading of non-critical resources until needed.

**Benefits:**
- Faster initial page load
- Reduced bandwidth usage
- Better Core Web Vitals (LCP)
- Saves resources for users who don't scroll

**When to use:**
- Images below the fold
- Heavy components not immediately visible
- Route-based code splitting
- Third-party widgets

**Q2: How does native lazy loading work?**
```html
<img src="image.jpg" loading="lazy" alt="Description" />
```
Browser automatically handles intersection detection and loading. Supported in modern browsers. For critical images use `loading="eager"`.

**Q3: Explain Intersection Observer API**
A: API that observes when elements enter/exit viewport.

```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      // Element is visible - load content
    }
  });
}, {
  rootMargin: '50px',  // Load 50px before visible
  threshold: 0.1        // 10% visible triggers
});
```

**Q4: How do you lazy load React components?**
```jsx
import { lazy, Suspense } from 'react';

const Heavy = lazy(() => import('./Heavy'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Heavy />
    </Suspense>
  );
}
```

**Q5: What's the difference between lazy loading and code splitting?**
A:
- **Lazy Loading**: Deferring resource loading until needed (images, components)
- **Code Splitting**: Breaking JavaScript bundle into smaller chunks
- Code splitting enables lazy loading of JavaScript

Both improve performance but target different resources.

**Q6: How would you implement infinite scroll?**
```jsx
const loaderRef = useRef(null);

useEffect(() => {
  const observer = new IntersectionObserver(([entry]) => {
    if (entry.isIntersecting) {
      loadMore();
    }
  });

  observer.observe(loaderRef.current);
  return () => observer.disconnect();
}, []);

return (
  <>
    {items.map(item => <Item key={item.id} />)}
    <div ref={loaderRef}>Loading...</div>
  </>
);
```

**Q7: What are the downsides of lazy loading?**
A:
- Delayed content visibility
- Complex implementation
- Need loading states/skeletons
- Can hurt SEO if not done properly
- Requires JavaScript (fallback needed)

**Solutions:**
- Server-side rendering for initial content
- Skeleton screens for better UX
- Progressive enhancement

**Q8: How do you lazy load images with blur-up effect?**
```jsx
const [src, setSrc] = useState(placeholder); // tiny base64

useEffect(() => {
  const img = new Image();
  img.onload = () => setSrc(fullImage);
  img.src = fullImage;
}, []);

<img src={src} style={{ filter: src === placeholder ? 'blur(10px)' : 'none' }} />
```

**Q9: What's content-visibility CSS property?**
```css
.section {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px;
}
```
Tells browser to skip rendering until visible. Similar to Intersection Observer but pure CSS. Improves rendering performance significantly.

**Q10: How do you test lazy loading?**
- Chrome DevTools Network tab: Verify resources load on scroll
- Lighthouse: Check for properly deferred resources
- Coverage tab: Check unused JavaScript
- Slow 3G throttling: Test loading states
- React DevTools Profiler: Check component render timing

## Summary

**Lazy Loading Strategies:**
1. **Images**: Use native `loading="lazy"` or Intersection Observer
2. **Components**: React.lazy() with Suspense
3. **Routes**: Dynamic imports with router
4. **Infinite scroll**: Intersection Observer on sentinel element

**Best Practices:**
- Eager load above-the-fold content
- Use appropriate rootMargin (100-200px)
- Implement skeleton screens
- Add error boundaries
- Preload on user intent (hover, route prediction)

**Performance Impact:**
- 20-40% faster initial load
- Significant bandwidth savings
- Better LCP scores
- Improved perceived performance

---

[← Core Web Vitals](./01-core-web-vitals.md) | [Next: Code Splitting →](./03-code-splitting.md)
