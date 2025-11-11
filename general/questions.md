# React app is loading 10 seconds for some user, why?

* Network latency: high RTT due to geographic distance to the server or CDN.
* No CDN edge nearby: static assets served from a single distant region.
* Large JS bundle: unoptimized React build, no code splitting or tree shaking.
* Uncompressed assets: missing gzip/brotli compression for JS/CSS.
* No HTTP/2 or HTTP/3: too many parallel requests with slow setup time.
* Slow DNS resolution: poor DNS provider or no global anycast.
* Backend latency: API responses slow due to inefficient queries or cold starts.
* Database bottlenecks: unindexed queries, connection pooling issues, or locking.
* Overloaded cluster nodes: K8s pods throttled (CPU/memory limits too low).
* Cold container start: pods scaling up slowly due to autoscaling or image pull.
* Uncached API responses: missing CDN, Redis, or HTTP caching layer.
* TLS handshake overhead: large cert chain or no TLS session reuse.
* Image size bloat: no WebP/AVIF compression or responsive sizing.
* Blocking scripts: synchronous JS or third-party trackers blocking render.
* No prerender/SSR: client renders everything, adding network + JS parse delay.
* Inefficient load balancer setup: traffic routed to wrong region or slow health checks.
* Database region mismatch: app and DB hosted in different datacenters.
* Logging or tracing overhead: excessive synchronous logging on critical path.
* Packet loss / unstable connection: mobile or satellite network instability.
* Missing browser caching headers: users redownload full assets each visit.
