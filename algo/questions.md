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