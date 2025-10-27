# In what order will console.log execute?

```js
const promise = new Promise((resolve, reject) => {
  console.log('1'); // First - synchronous code in Promise constructor
  resolve();
  console.log('2'); // Second - synchronous code in Promise constructor
});

console.log('3'); // Third - synchronous code

setTimeout(() => console.log('7'), 0); // Seventh - macrotask queue

console.log('4'); // Fourth - synchronous code

promise.then(() => console.log('6')); // Sixth - microtask queue

console.log('5'); // Fifth - synchronous code
```

**Execution Order:**
1. Promise constructor (synchronous)
2. Promise constructor (synchronous) 
3. Synchronous code
4. Synchronous code
5. Synchronous code
6. Promise .then() - microtask
7. setTimeout - macrotask


# Create you own getUrl

```ts
/**
 * Fetches data from a URL with retry logic and timeout handling
 * 
 * @param url - The URL to fetch data from
 * @param retries - Number of retry attempts if the request fails (e.g., 3 for 3 retries)
 * @param timeout - Request timeout in milliseconds before considering it failed (e.g., 5000 for 5 seconds)
 * @param delay - Delay in milliseconds between retry attempts (e.g., 1000 for 1 second delay)
 * @returns Promise that resolves to Response object on success
 */
function getUrl(url: string, retries: number, timeout: number, delay: number): Promise<Response>;
```

```js
// Async/Await approach with loop
async function getUrl(url, retries, timeout, delay) {
  for (let attempt = 0; attempt <= retries; attempt++) {
    try {
      // Create timeout promise that rejects after specified time
      const timeoutPromise = new Promise((_, reject) => {
        setTimeout(() => reject(new Error('Request timeout')), timeout);
      });
      
      // Race between fetch and timeout
      const response = await Promise.race([
        fetch(url),
        timeoutPromise
      ]);
      
      // Check if response is ok (status 200-299)
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      return response;
      
    } catch (error) {
      console.log(`Attempt ${attempt + 1} failed:`, error.message);
      
      // If this was the last attempt, throw the error
      if (attempt === retries) {
        throw error;
      }
      
      // Wait before next retry
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Promise-based recursive approach
function getUrlRecursive(url, retries, timeout, delay, attempt = 0) {
  // Create timeout promise
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => reject(new Error('Request timeout')), timeout);
  });
  
  // Race between fetch and timeout
  return Promise.race([
    fetch(url),
    timeoutPromise
  ])
  .then(response => {
    // Check if response is ok
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    return response;
  })
  .catch(error => {
    console.log(`Attempt ${attempt + 1} failed:`, error.message);
    
    // If no more retries, reject with the error
    if (attempt >= retries) {
      return Promise.reject(error);
    }
    
    // Wait and retry recursively
    return new Promise(resolve => setTimeout(resolve, delay))
      .then(() => getUrlRecursive(url, retries, timeout, delay, attempt + 1));
  });
}

// Alternative without Promise.race() - Manual timeout handling
function getUrlWithoutRace(url, retries, timeout, delay, attempt = 0) {
  return new Promise((resolve, reject) => {
    let isResolved = false;
    
    // Set up timeout
    const timeoutId = setTimeout(() => {
      if (!isResolved) {
        isResolved = true;
        reject(new Error('Request timeout'));
      }
    }, timeout);
    
    // Make the fetch request
    fetch(url)
      .then(response => {
        if (!isResolved) {
          isResolved = true;
          clearTimeout(timeoutId);
          
          if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
          }
          resolve(response);
        }
      })
      .catch(error => {
        if (!isResolved) {
          isResolved = true;
          clearTimeout(timeoutId);
          reject(error);
        }
      });
  })
  .catch(error => {
    console.log(`Attempt ${attempt + 1} failed:`, error.message);
    
    if (attempt >= retries) {
      throw error;
    }
    
    // Wait and retry
    return new Promise(resolve => setTimeout(resolve, delay))
      .then(() => getUrlWithoutRace(url, retries, timeout, delay, attempt + 1));
  });
}

// AbortController approach (modern alternative)
async function getUrlWithAbort(url, retries, timeout, delay) {
  for (let attempt = 0; attempt <= retries; attempt++) {
    const controller = new AbortController();
    
    // Set up timeout that aborts the request
    const timeoutId = setTimeout(() => controller.abort(), timeout);
    
    try {
      const response = await fetch(url, {
        signal: controller.signal 
      });
      
      clearTimeout(timeoutId);
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      return response;
      
    } catch (error) {
      clearTimeout(timeoutId);
      console.log(`Attempt ${attempt + 1} failed:`, error.message);
      
      if (attempt === retries) {
        throw error;
      }
      
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

```


# Explain useLayoutEffect

## What is useLayoutEffect?

`useLayoutEffect` is a React Hook that runs synchronously **before** the browser paints the screen, unlike `useEffect` which runs **after** the paint.

## Key Differences from useEffect

| Aspect | useEffect | useLayoutEffect |
|--------|-----------|----------------|
| **Timing** | After DOM mutations + paint | After DOM mutations, before paint |
| **Execution** | Asynchronous | Synchronous |
| **Performance** | Non-blocking | Can block painting |
| **Use Case** | Side effects, API calls | DOM measurements, synchronous updates |

## When to Use useLayoutEffect

### ✅ Good Use Cases:
1. **DOM measurements** - Reading element dimensions, positions
2. **Synchronous DOM mutations** - Preventing visual flicker
3. **Scroll position restoration**
4. **Focus management**

### ❌ Avoid for:
- API calls
- Subscriptions  
- Most side effects
- Heavy computations

## Interview Examples

```jsx
// ❌ BAD: This causes flicker
function BadExample() {
  const [width, setWidth] = useState(0);
  const divRef = useRef();

  useEffect(() => {
    // Runs AFTER paint - user sees flash of wrong width
    setWidth(divRef.current.offsetWidth);
  }, []);

  return <div ref={divRef}>Width: {width}px</div>;
}

// ✅ GOOD: No flicker
function GoodExample() {
  const [width, setWidth] = useState(0);
  const divRef = useRef();

  useLayoutEffect(() => {
    // Runs BEFORE paint - no visual flash
    setWidth(divRef.current.offsetWidth);
  }, []);

  return <div ref={divRef}>Width: {width}px</div>;
}

// Real-world example: Tooltip positioning
function Tooltip({ children, content }) {
  const [position, setPosition] = useState({ top: 0, left: 0 });
  const tooltipRef = useRef();

  useLayoutEffect(() => {
    if (tooltipRef.current) {
      const rect = tooltipRef.current.getBoundingClientRect();
      
      // Calculate position to keep tooltip in viewport
      setPosition({
        top: rect.bottom + 10,
        left: Math.max(0, rect.left - 100)
      });
    }
  });

  return (
    <div ref={tooltipRef}>
      {children}
      <div 
        className="tooltip" 
        style={{ 
          position: 'absolute',
          top: position.top,
          left: position.left 
        }}
      >
        {content}
      </div>
    </div>
  );
}
```

## Interview Talking Points

1. **Performance Impact**: useLayoutEffect blocks painting, so use sparingly
2. **Visual Consistency**: Prevents layout shifts and flicker
3. **Synchronous Nature**: Code runs immediately, can impact performance
4. **Server-Side Rendering**: Both hooks don't run on server, but useLayoutEffect warns in development

## Quick Decision Guide

**Use useLayoutEffect when:**
- You need to read DOM properties immediately
- Visual changes must be synchronous
- Preventing flicker is critical

**Use useEffect for everything else!**


# Implement useDebounce

```ts
function useDebounce(cb: T, delay: number): T;
```

## Implementation (stable reference + cleanup)

```ts
import { useEffect, useMemo, useRef } from 'react';

export function useDebounce<T extends (...args: any[]) => any>(cb: T, delay: number): T {
  const cbRef = useRef(cb);
  const delayRef = useRef(delay);
  const timeoutRef = useRef<ReturnType<typeof setTimeout> | null>(null);

  cbRef.current = cb;
  delayRef.current = delay;

  useEffect(() => {
    return () => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
    };
  }, []);

  const debounced = useMemo(() => {
    const fn = (...args: Parameters<T>) => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
      timeoutRef.current = setTimeout(() => {
        cbRef.current(...args);
      }, delayRef.current);
    };

    return fn as unknown as T;
  }, []);

  return debounced;
}
```

# Give some examples of Tasks and Microtasks

Concise cheat‑sheet for interviews:

- Order: run current Task (macrotask) → drain all Microtasks → render → next Task.

Examples in browsers
- Tasks (macrotasks):
  - setTimeout, setInterval
  - DOM event callbacks (click, input, etc.)
  - network/I/O callbacks, message events (postMessage/MessageChannel)
  - script execution on page load
- Microtasks:
  - Promise.then/catch/finally and async/await continuations
  - queueMicrotask
  - MutationObserver callbacks

Examples in Node.js
- Tasks (macrotasks):
  - setTimeout, setInterval, setImmediate
  - I/O callbacks (fs, net, http)
- Microtasks:
  - Promise.then/catch/finally
  - process.nextTick (runs before other microtasks)

Rule of thumb: microtasks always run to completion right after the current task finishes and before timers or I/O handlers of the next task.

