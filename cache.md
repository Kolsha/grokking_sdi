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
CDNs are a kind of cache.


**4. When do we choose to use CDN ?**  
CDN is a distributed network of proxy servers.  
CDN can be used to store **static media**.


**5. How does CDN work ?**  
A request asks CDN for a static media --> CDN checks if the data is locally available --> if yes, send the data back to the requesting server. if not, query the back-end servers for the file, cache it locally, send the data to the requesting user.  
**The basic idea of CDN: CDNs place Data Centers at strategic locations across the globe.**

**6. What if the cache is out of date (is not coherent/consistent with the source of truth (e.g. database) ?**  
Cache Invalidation:
- **Write-through cache**: when data is written into database, write the cache as well.
- **Write-around cache**: data is written into database directly, bypassing cache. When a new read request is sent to the request layer, there will be a cache miss. Then the request layer needs to read from slower back-end storage, update the cache.
- **Write-back cache**: data is writen to cache alone. After specified intervals or certain conditions, write to permanent storage.

**7. How to free the memory for old or unused data in cache (i.e. cache eviction)?** ?
- FIFO
- LIFO
- LRU
- MRU
- LFU
- Random Replacement
