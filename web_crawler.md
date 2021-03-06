## Web Crawler

**1. What is web crawler ?**  
Web crawler collects documents by **recursively** fetching links from **a set of starting pages**.

**2. Uses of web crawler**  
- search engine downloads/collects all the pages, in order to create an index on them to perform faster searches;
- search for copyright infringements;
- monitor sites to see when their structure or contents change;

**3. Requirements and Goals of the System**
- scalability
- **extensibility**: service should be designed in **a modular way** with the expectation that new functionality will be added to it (such as there could be newer document types that need to be downloaded and processed in the future).

**4. Some Design Considerations**
- only fetch HTML pages ? Or fetch other types of media such as sound files, images, videos ?
    - **we need different sets of parsing modules for different media type, one for HTML, another for images, or another for videos ...**
- only consider HTTP protocols ? how about other protocols such as FTP links ?
    - **It shouldn't be hard to extend the design to use FTP or other protocols**
- how big will the URL database be?
    - **If we crawl _1 billion websites_, let's assume _an upper bound of 15 billion different web pages_ that will be reached by our crawler.** 
- How to deal with Robots Exclusion Standard ?
    - **What is Robots Exclusion Standard ?**
      - It is a standard used by websites to communicate with web crawlers (i.e. web robots).
      - It specifies what areas of the website(i.e. page) crawlers are allowed to download.
      - It also specifies which areas of the website(i.e. page) crawlers are not allowed to download/process/scan/crawl.
    - Before web crawler attempts to crawl a website, it should first download the robots.txt file such as https://www.amazon.com/robots.txt .

**5. Capacity Estimation and Constraints**  
**Storage Estimation**
- needs to crawl 15 billion pages within 4 weeks
- average web page(HTML page) has size ~= 100KB
- for every web page, **we are storing 500 bytes of metadata**, such as size, creationDate, ...
- assume a 70% capacity model
- **In total, we need (15 billion * (100KB + 0.5KB)) / 0.7 = 2250 TB = 2.25PB /petabytes.**
- **needs to crawl 15 billion / (4 weeks * 7days * 24hours * 3600) = 6200 pages/s**

**6. High level design**
- **Prerequisite Leetcode**  
Try to implement them in both BFS and DFS.
    - [Web Crawler](https://leetcode.com/problems/web-crawler/)
    - [Web Crawler Multithreaded](https://leetcode.com/problems/web-crawler-multithreaded/)
    
**7. Detailed Component design**
- **BFS or DFS crawling**
    - DFS is usually not a good choice because of its deep depth
- **Path-ascending crawling**
    -  What if there is no inbound link can be found in a website ? Path-ascending crawler would **ascend to every path** in the URL. For example, for http://foo.com/a/b/c. The path-ascending crawler crawls http://foo.com/a first, then http://foo.com/a/b, then http://foo.com/a/b/c.

- **Difficulties in implementing an efficient web crawler**
    - large volume of web pages
    - rate of change on web pages

- **What is URL frontier ? Why do we need URL frontier ?**  
    - **Problems (here are why we need URL frontier)**:
        - **politeness**: a web crawler should avoid sending too many requests to the same hosting server within a short period.
        - **prioritize URLs to be downloaded**: a web page from a discussion forum about Apple product vs. a web page on the Apple home page
    - **URL frontier does the following:**
        - store URLs to be downloaded; and
        - prioritize which URLs should be crawled/downloaded first; and
        - ensure politeness
            - **How ?**
                - maintain a mapping from _website hostname_ to _queue which only contains URLs from this website hostname._
                - maintain a mapping from _worker thread_ to a _queue_ which contains URLs (from the same website hostname).
                - For every worker thread, **add a delay between 2 consecutive download tasks.**
        - **Problem: URL frontier cannot store millions of URLs**
            - store all URLs on disk ? No. reading will be slow. 
            - store all URLs on memory ? No. not scalable.
            - **hybrid**: The majority of URLs are stored on disk. And we maintain buffers in memory for enqueue/dequeue operations.
                - for a new extracted URL, enqueue this URL to a enqueue_buffer (which is implemented a queue); When this enqueue_buffer is full, it dumps all its URLs to disk.
                - to retrieve 1 URL from URL frontier, dequeue a URL from a dequeue_buffer (which is implemented as a queue). This dequeue_buffer periodically read URLs from disk.

- **Downloader/Fetcher**
    - Implement it in a modular way for extensibility, so to support HTTP protocol, FTP protocol and other protocols.
    - obey Robot Exclusion Standard.
    - **The bottleneck for a webcrawler: DNS Resolving**
        - DNS reponse time ranges from 10ms to 200ms.
        - **How to solve this problem ?** Maintain a DNS cache which keeps the domain name to IP address mapping.

- **Document input stream (DIS)**
    - What is DIS ?
        - **cache** the entire contents of the document read from the internet  
    - Why need DIS ?
        - to enable same document to be processed by multiple processing modules (i.e. **provide method to re-read the document**).

- **Document Dedupe test**
    - How to deduplicate ?
        - store chechsum of every processed document into a database;
        - for every new document, compute its checksum, and see if this checksum is contained in the database.
        - Where to store this database ?
            - depends on the storage we need. If it is big, such as 1TB, you can store LRU cache in memory and store other checksums in a persistent storage (you can call it back storage or disk).

- **URL filters**
    - it is used to block websites so that our crawler can ignore them.

- **URL Dedupe test**
    - How to deduplicate ? The idea is the same as **Document Dedupe test**.
    - **One more thing to highlight**: 
        - we can keep an in-memory cache of **popular URLs** on each host **shared by all threads**, **in order to lead to a high in-memory hit rate**.

**8. Fault Tolerance**
- What if the web crawler server crashes ?
    - In this case, **consistent hashing** helps, because it can shift the load(i.e. URL stored on the disk, URL checksum set, document store, and document checksum set) of the crashing server to other servers. 

**9. Perform Checkpointing**
- What is performing checkpointing ?
    - each host performs checkpointing periodically by **dumping a snapshot of all the data it is holding onto a remote server**. 
- What is the benefit of performing checkpointing ?
    - if a server performs checkpointing, when this server dies, another server can replace it by taking its data from the **last snapshot**. 

**10. Crawler Trap**
- What is crawler trap ?
    - it is a URL or set of URLs that cause a crawler to crawl indefinitely. 

**11. High-level design**
![web_crawler](https://user-images.githubusercontent.com/26174882/155469610-a2aecd5d-bc59-482d-83d0-0780f3ff1564.png)
