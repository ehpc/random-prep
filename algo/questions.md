# Look at figure points-cluster.png. How to find the densest cluster of points?

Use DBSCAN (Density-Based Spatial Clustering of Applications with Noise).

**Build cluster**

```python
def clusters_1d(xs, d_max):
    xs = sorted(xs)
    clusters = []
    current = [xs[0]]

    for i in range(1, len(xs)):
        if xs[i] - xs[i-1] <= d_max:
            current.append(xs[i])
        else:
            clusters.append(current)
            current = [xs[i]]
    clusters.append(current)
    return clusters
```

**Pick the densest cluster**

```python
def densest_cluster(xs, d_max, min_points=2):
    for_x = clusters_1d(xs, d_max)
    best = None
    best_density = -1

    for C in for_x:
        if len(C) < min_points:
            continue
        span = C[-1] - C[0]
        if span == 0:
            continue
        density = len(C) / span
        if density > best_density:
            best_density = density
            best = C

    return best, best_density
```

# 

```ts
// Counts primitive values recursively
function flatCount(data: any[]) {
    return 0;
}

flatCount([1, 2]); // 2
flatCount([1, [2, 3, [4, 6]]]); // 5
let arrA = [4, 5];
arrA.push(arraA);
flatCount([1, arrA]);
```

```ts
function flatCount(data: any[], seen = new Set()): number {
  if (seen.has(data)) return 0;
  seen.add(data);

  let count = 0;

  for (const item of data) {
    if (Array.isArray(item)) {
      count += flatCount(item, seen);
    } else {
      count += 1;
    }
  }

  return count;
}
```

# Construct ATM machine algorithm

```js
let limits = {
    5000: 5,
    1000: 5,
    500: 5,
    100: 5,
    23: 13,
    543: 1,
    3: 5
};
// limits = {
//     5: 1,
//     3: 2,  
// };
// atm(6, limits)

console.log('atm result:', atm(15600, limits)); // return { 5000: 3, 500: 1, 100: 1 }
console.log('limits:', limits);

console.log('atm result:', atm(15600, limits)); // return { 5000: 2, 1000: 5, 500: 1, 100: 1 }
console.log('limits:', limits);

console.log('atm result:', atm(24, limits)); // log error, don't modify limits
console.log('limits:', limits);

function atm(sum, limits) {
    const banknotes = Object.keys(limits).map(Number).sort((a, b) => b - a);
    let result;

    function dfs(banknoteIndex, remainingSum, usedBanknotes) {
        if (remainingSum === 0) {
            result = {...usedBanknotes};
            return true;
        } else if (remainingSum < 0) {
            return false;
        }
        if (banknoteIndex > banknotes.length - 1) {
            return false;
        }
        
        const banknote = banknotes[banknoteIndex];
        const maxBanknotes = limits[banknote];
        const neededBanknotes = Math.floor(remainingSum / banknote);
        const upperLimit = Math.min(neededBanknotes, maxBanknotes);
        for (let banknoteCount = upperLimit; banknoteCount >= 0; banknoteCount--) {
            const currentSum = banknoteCount * banknote;
            const currentlyUsedBanknotes = banknoteCount > 0 ? {
                [banknote]: banknoteCount,
            } : {};
            if (banknoteCount) {
                usedBanknotes[banknote] = banknoteCount;
            } else {
                delete usedBanknotes[banknote];
            }
            const shouldStop = dfs(banknoteIndex + 1, remainingSum - currentSum, usedBanknotes);
            if (shouldStop) return true;
        }
        return false;
    }

    dfs(0, sum, {});

    if (result) {
        for (const [banknote, count] of Object.entries(result)) {
            limits[banknote] -= count;
        }
    }

    return result;
}

```

# Sort plane tickets

```js
const tickets = [
  { from: 'Berlin', to: 'Tokyo' },
  { from: 'London', to: 'Berlin' },
  { from: 'Paris', to: 'Madrid' },
  { from: 'Tokyo', to: 'Seoul' },
  { from: 'Madrid', to: 'London' },
];

const sortedTickets = [
  { from: 'Paris', to: 'Madrid' },
  { from: 'Madrid', to: 'London' },
  { from: 'London', to: 'Berlin' },
  { from: 'Berlin', to: 'Tokyo' },
  { from: 'Tokyo', to: 'Seoul' },
];

// Space complexity: not limited
// Time complexity: O(n)
function sortTickets(tickets) {
    if (tickets.length === 0) return [];

    const toByFrom = new Map();
    const toSet = new Set();

    for (const {from, to} of tickets) {
        toByFrom.set(from, to);
        toSet.add(to);
    }

    let startCity = toByFrom.keys().find((city) => !toSet.has(city));
    const sorted = [];
    let from = startCity;
    while (toByFrom.has(from)) {
        const to = toByFrom.get(from);
        sorted.push({ from, to });
        from = to;
    }
    return sorted;
}

```
