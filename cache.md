## Caching

**1. What is cache ?**
- **The Locality of reference principle**: recently requested data is likely to be requested again.
- It's like a short-term memory
    - it has a limited amount of space;
    - it contains recently accessed items;
    - caches is often found at the level nearest to the front end, _so that data can be returned without taxing downstream levels_.

**2. Where is the cache stored ?**
Can be stored in memory or local disk (which is faster than going to network storage).
- What if the request layer is expanded into multiple nodes ?
    - the same request, distributed by load balancer, will go to different nodes, thus increasing cache **misses**.
    - **How to fix it ?** Use global caches and distributed caches.

**3. Cache vs. CDN**
- CDNs are a kind of cache.

**4. When do we choose to use CDN ?**  
CDN can be used to store **static media**.

**5. How does CDN work ?**  
A request asks CDN for a static media --> CDN checks if the data is locally available --> if yes, send the data back to the requesting server. if not, query the back-end servers for the file, cache it locally, send the data to the requesting user.  
**The basic idea of CDN: CDNs place Data Centers at strategic locations across the globe.**

**6. **
