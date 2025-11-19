1. Look at figure points-cluster.png. How to find the densest cluster of points?

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
