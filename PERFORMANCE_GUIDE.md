# Performance Optimization Guide

A comprehensive guide to identifying and improving slow or inefficient code.

---

## 📊 Table of Contents

1. [Identifying Performance Issues](#identifying-performance-issues)
2. [Common Performance Anti-Patterns](#common-performance-anti-patterns)
3. [Optimization Strategies](#optimization-strategies)
4. [Language-Specific Tips](#language-specific-tips)
5. [Monitoring and Profiling](#monitoring-and-profiling)

---

## 🔍 Identifying Performance Issues

### Signs of Performance Problems

- **Slow Response Times**: Application takes too long to respond to user actions
- **High CPU Usage**: Processes consuming excessive CPU resources
- **Memory Leaks**: Memory usage growing over time without being released
- **Blocking Operations**: UI freezes or becomes unresponsive
- **Database Bottlenecks**: Slow queries or excessive database calls

### Profiling Tools

**JavaScript/TypeScript:**
- Chrome DevTools Performance tab
- Node.js built-in profiler
- `clinic.js` for Node.js applications
- React DevTools Profiler

**General:**
- Application Performance Monitoring (APM) tools
- Custom timing measurements with `performance.now()`
- Network monitoring tools

---

## ⚠️ Common Performance Anti-Patterns

### 1. N+1 Query Problem

**Problem:**
```javascript
// Bad: Makes N+1 database queries
async function getUsersWithPosts() {
  const users = await db.users.findAll();
  for (const user of users) {
    user.posts = await db.posts.findByUserId(user.id); // N queries
  }
  return users;
}
```

**Solution:**
```javascript
// Good: Single query with proper joins or eager loading
async function getUsersWithPosts() {
  return await db.users.findAll({
    include: [{ model: db.posts }]
  });
}
```

### 2. Inefficient Loops and Iterations

**Problem:**
```javascript
// Bad: Multiple array iterations
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(n => n * 2);
const filtered = doubled.filter(n => n > 4);
const sum = filtered.reduce((acc, n) => acc + n, 0);
```

**Solution:**
```javascript
// Good: Single pass through the array
const sum = numbers.reduce((acc, n) => {
  const doubled = n * 2;
  return doubled > 4 ? acc + doubled : acc;
}, 0);
```

### 3. Unnecessary Re-renders (React)

**Problem:**
```javascript
// Bad: Creates new object on every render
function UserProfile({ user }) {
  const style = { color: 'blue', fontSize: '16px' };
  return <div style={style}>{user.name}</div>;
}
```

**Solution:**
```javascript
// Good: Memoize or move outside component
const STYLE = { color: 'blue', fontSize: '16px' };

function UserProfile({ user }) {
  return <div style={STYLE}>{user.name}</div>;
}

// Or use useMemo for dynamic values
function UserProfile({ user, isActive }) {
  const style = useMemo(
    () => ({ color: isActive ? 'blue' : 'gray', fontSize: '16px' }),
    [isActive]
  );
  return <div style={style}>{user.name}</div>;
}
```

### 4. Synchronous Blocking Operations

**Problem:**
```javascript
// Bad: Blocks event loop
function processData(data) {
  const result = heavyComputation(data); // Blocks for seconds
  return result;
}
```

**Solution:**
```javascript
// Good: Use async/await or Web Workers
async function processData(data) {
  return await processInWorker(data);
}

// Or break into chunks
function processDataInChunks(data) {
  return new Promise((resolve) => {
    const chunks = splitIntoChunks(data);
    let results = [];
    
    function processNext(index) {
      if (index >= chunks.length) {
        resolve(results);
        return;
      }
      
      setTimeout(() => {
        results.push(processChunk(chunks[index]));
        processNext(index + 1);
      }, 0);
    }
    
    processNext(0);
  });
}
```

### 5. Memory Leaks

**Problem:**
```javascript
// Bad: Event listeners not cleaned up
class Component {
  constructor() {
    window.addEventListener('resize', this.handleResize);
  }
  
  handleResize() {
    // Handle resize
  }
}
```

**Solution:**
```javascript
// Good: Clean up listeners
class Component {
  constructor() {
    this.handleResize = this.handleResize.bind(this);
    window.addEventListener('resize', this.handleResize);
  }
  
  handleResize() {
    // Handle resize
  }
  
  destroy() {
    window.removeEventListener('resize', this.handleResize);
  }
}
```

---

## 🚀 Optimization Strategies

### 1. Caching

**Memoization:**
```javascript
// Cache expensive function results
const memoize = (fn) => {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
};

const expensiveCalculation = memoize((n) => {
  // Heavy computation
  return n * n;
});
```

**HTTP Caching:**
```javascript
// Set appropriate cache headers
res.setHeader('Cache-Control', 'public, max-age=3600');
```

### 2. Lazy Loading

**Code Splitting (React):**
```javascript
import React, { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <HeavyComponent />
    </Suspense>
  );
}
```

**Image Lazy Loading:**
```html
<img src="image.jpg" loading="lazy" alt="Description" />
```

### 3. Debouncing and Throttling

**Debounce (wait for pause):**
```javascript
function debounce(fn, delay) {
  let timeoutId;
  return (...args) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn(...args), delay);
  };
}

// Usage: Search input
const searchHandler = debounce((query) => {
  performSearch(query);
}, 300);
```

**Throttle (limit frequency):**
```javascript
function throttle(fn, limit) {
  let inThrottle;
  return (...args) => {
    if (!inThrottle) {
      fn(...args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}

// Usage: Scroll handler
const scrollHandler = throttle(() => {
  handleScroll();
}, 100);
```

### 4. Use Efficient Data Structures

**Choose the Right Structure:**
```javascript
// Bad: Using array for lookups (O(n))
const users = [{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }];
const user = users.find(u => u.id === 2);

// Good: Using Map for fast lookups (O(1))
const users = new Map([
  [1, { id: 1, name: 'Alice' }],
  [2, { id: 2, name: 'Bob' }]
]);
const user = users.get(2);
```

**Use Set for Uniqueness:**
```javascript
// Bad: Array includes (O(n))
const uniqueItems = [];
items.forEach(item => {
  if (!uniqueItems.includes(item)) {
    uniqueItems.push(item);
  }
});

// Good: Set (O(1))
const uniqueItems = [...new Set(items)];
```

### 5. Optimize Algorithms

**Time Complexity Matters:**
```javascript
// Bad: O(n²) nested loop
function findDuplicates(arr) {
  const duplicates = [];
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] === arr[j]) {
        duplicates.push(arr[i]);
      }
    }
  }
  return duplicates;
}

// Good: O(n) with Set
function findDuplicates(arr) {
  const seen = new Set();
  const duplicates = new Set();
  
  arr.forEach(item => {
    if (seen.has(item)) {
      duplicates.add(item);
    }
    seen.add(item);
  });
  
  return [...duplicates];
}
```

---

## 💻 Language-Specific Tips

### JavaScript/TypeScript

1. **Avoid `delete` operator**: Use `Map` or set to `undefined` instead
2. **Use `const` and `let`**: Avoid `var` for better optimization
3. **Prefer `for...of` over `forEach`**: Slightly faster for large arrays
4. **Use `requestAnimationFrame` for animations**: Better than `setTimeout`
5. **Batch DOM updates**: Minimize reflows and repaints

### React

1. **Use React.memo**: Prevent unnecessary re-renders
2. **useCallback and useMemo**: Memoize functions and values
3. **Virtual scrolling**: For long lists (react-window, react-virtualized)
4. **Key prop optimization**: Use stable, unique keys
5. **Avoid inline functions in JSX**: Can cause unnecessary re-renders

### Next.js

1. **Use Static Site Generation (SSG)**: When possible
2. **Optimize images**: Use `next/image` component
3. **Dynamic imports**: Code splitting for routes and components
4. **API routes optimization**: Implement caching strategies
5. **Font optimization**: Use `next/font` for better performance

---

## 📈 Monitoring and Profiling

### Performance Metrics to Track

**Web Vitals:**
- **LCP (Largest Contentful Paint)**: < 2.5s
- **FID (First Input Delay)**: < 100ms
- **CLS (Cumulative Layout Shift)**: < 0.1

**Custom Metrics:**
```javascript
// Measure specific operations
const start = performance.now();
performOperation();
const end = performance.now();
console.log(`Operation took ${end - start}ms`);

// Mark and measure
performance.mark('start-operation');
performOperation();
performance.mark('end-operation');
performance.measure('operation-duration', 'start-operation', 'end-operation');
```

### Continuous Monitoring

1. **Set up APM tools**: New Relic, DataDog, or similar
2. **Implement error tracking**: Sentry, Rollbar
3. **Monitor bundle size**: webpack-bundle-analyzer
4. **Track Core Web Vitals**: Google Search Console, Lighthouse CI
5. **Set performance budgets**: Fail builds that exceed thresholds

---

## 🎯 Best Practices Checklist

- [ ] Profile before optimizing (measure first!)
- [ ] Focus on bottlenecks (80/20 rule applies)
- [ ] Write readable code first, optimize later
- [ ] Use production builds for performance testing
- [ ] Implement caching at multiple levels
- [ ] Minimize network requests
- [ ] Optimize images and assets
- [ ] Use CDN for static assets
- [ ] Enable gzip/brotli compression
- [ ] Implement proper error handling
- [ ] Monitor production performance
- [ ] Set up performance budgets
- [ ] Regularly audit dependencies
- [ ] Remove unused code

---

## 📚 Additional Resources

### Tools
- [Lighthouse](https://developers.google.com/web/tools/lighthouse)
- [WebPageTest](https://www.webpagetest.org/)
- [Chrome DevTools](https://developer.chrome.com/docs/devtools/)
- [React DevTools Profiler](https://react.dev/reference/react/Profiler)

### Reading
- [Web Performance Working Group](https://www.w3.org/webperf/)
- [High Performance Browser Networking](https://hpbn.co/)
- [JavaScript Performance Optimization](https://developers.google.com/web/fundamentals/performance/rendering)

---

## 💡 Remember

> "Premature optimization is the root of all evil." - Donald Knuth

Always:
1. **Measure** before optimizing
2. **Profile** to find actual bottlenecks
3. **Optimize** what matters most to users
4. **Test** to ensure improvements
5. **Monitor** in production

---

*Keep building efficiently! 🚀*
