## API Rate Limiter

**1. What is a Rate Limiter ?**
- **Problem:** a service, which can only serve a limited number of requests per second, is receiving a huge number of requests.
- **Goal:** To solve this problem, we need some kind of throttling or rate limiting mechanism that would allow only a certain number of requests so our service can respond to all of them.
- **Rate Limiter:** limits the number of events **an entity (user, device, IP, etc.)** can perform **in a particular time window**.
- **API Rate Limiter:** Limit the number of requests an entity can send to an API(or API server) within a time window.

**2. Benefits of Rate Limiter ?**
- Prevents services from
  - **Denial-of-service(DOS)** attacks;
  - brute-force password attempts, brute-force credit card transactions, etc;
  - misbehaving clients/scripts sending a lot of lower-priority requests, such as requests for analytics;
  - bad design practices(sloppy development tactics) such as requesting the same information over and over again;
- Create a revenue model based on rate limiting - the user has to buy higher limits.

**3. Requirement clarification**
- Functional Requirements:
  - **The user should get an error message _"HTTP status 429 - Too many requests"_** whenever the defined threshold is crossed within a single server or across a combination of servers.
- Non-Functional Requirements
  - highly available.
  - do not introduce substantial latencies.

**4. Throttling can be defined in application level and/or API level**  
3 types of throttling:
- hard throttling;
- soft throttling;
- elastic or dynamic throttling: if the system has some resources available, the # of requests can be > the threshold.

**5. Algorithms used for Rate Limiting**
- Fixed Window Algorithm: 
  - definition of the time window: from the start of the **time-unit** to the end of **the time-unit**.  
  - the time window is _**irrespective** of the time frame at which the API request has been made_.
- Rolling Window Algorithm:
  - definition of the time window: from the fraction of the time at which the request is made + the time window length. For example, if a message is sent at 300th millisecond of a second. The time window is from _the 300th millisecond of that second_ to _the 300th millisecond of next second._

**6. High level design**  
The key: **web server** first asks the Rate Limiter to decide if it will be throttled or served. If it can be served, **the web server** will pass the request to API server.
![rate_limiter](https://user-images.githubusercontent.com/26174882/151849772-5be8dcae-c2ff-4d3a-961d-561c0f712cfa.png)

**7. Detailed Design: Basic System Design and Algorithm**

```
# Each user --> 
#              count (representing how many requests the user has made within a time window), and
#              timestamp (at which we started counting the requests. It's NORMALIZED, FOR EXAMPLE, TO MINUTE !!!)
# {UserID : (count, startTime)}
 
 for request from a user:
    if user in hash_table:
        count, startTime = hash_table[user]
        currentTime = getTimeStamp() # It is normalized! For example, it's normalized to minute.
        if currentTime - startTime >= time_window: # such as 1 min
            count = 1
            startTime = currentTime
        else:
            if count >= threshold: # such as 3
                reject the request
            else:
                count += 1
                allow the request
    else:
        startTime = getTimeStamp()
        count = 1
        hash_table[userID] = (count, startTime)
        allow the request
```

**Dive deep into details: What are some of the problems with the algorithm above ?**  
- This is a fixed window algorithm.
  - **How to fix it?** Use Sliding window algorithm.
- In a distributed environment, "Read-and-then-write" behavior can create a **race condition** (i.e. a computer program depends on the sequence or timing of the program's processes/threads).
  - **How to fix it?**
    - For Redis (an **in-memory** data structure store, used as a distributed, in-memory **key-value database**), use **Redis Lock** for the duration of read-and-then-write algorithm.
    - Memcached: a general-purposed distributed memory-caching system. It caches data and objects in RAM. Memcached's **APIs provide a very large hash table distributed across multiple machines**.

**Back-of-the-envelop estimation: How much storage do we need to store all of the user data ?**  
(8 + 2 + 2 + 20 + 4) bytes * 1,000,000 = 36,000 KB = 36MB for 1 million user
- UserID: 8 Bytes (1 byte per char);
- Count: 2 Bytes;
- StartTime: 2 Bytes;
- **Hash-table** has an overhead of 20 bytes for each record.
- For each user, we need 4 byte **lock** to resolve atomicity problem.
- Assume 1 million users.

**Sliding Window Algorithm**
```
# for every user, a Redis Sorted Set is used to store timestamps of each request sent by this user

for every request sent by a user:
    sortedTimeStamp = getSortedSetForUser(userID)
    remove timestamps, for which timestamp - currentTimeStamp > time_window, from the sortedTimeStamp
    if size of sortedTimeStamp > threshold:
        reject this request
    else:
        add currentTimeStamp into the sortedSet
        accept this request
```

**How much storage will we need if sliding window algorithm is used?**
8 + (4 + 20) * 500 + 20 = 12KB per user
12 KB * 1,000,000 = 12,000MB = 12GB for 1 million users
- UserID: 8 bytes
- Every timestamp: 4 bytes
- Threshold (i.e. a rate limiting of 500 requests per hour): 500 requests per hour
- 20 bytes overhead for Redis Sorted Set
- 20 bytes overhead for hash table

**Sliding Window With Counters**
A weighted count(i.e. counter) in previous window + the count(i.e. counter) in current window
```
for every request from a user:
    current_unnormalized_timestamp = getCurrentUnNormalizedTimeStamp()
    current_timestamp = getCurrentTimeStamp() # NORMALIZED! For example, it's normalized to minute.
    prev_timestamp = current_timestamp - time_window
    
    weight = (current_unnormalized_timestamp - current_timestamp) / time_window
    prev_count = hash_table[prev_timestamp]
    curr_count = hash_table.get(current_timestamp, 0) + 1
    
    count = prev_count * weight + curr_count
    if count <= threshold:
        accept
    else
        reject
```
**Note**: use _RedisHash_ as data structure of hash table. Every key has to TTL (i.e. the key will expire after TTL).

**How much storage will we need if sliding window with counters algorithm is used ?**

**8. Should we rate limit by IP or by user ?**
- cons of IP based rate limiting
  - When multiple users share a single public IP like in an internet cafe or smartphone users that are using the same gateway, 1 bad user can cause throttling to other users.
  - There are a huge number of IPv6 addresses available to a hacker from even one computer. This can make server run out of memory tracking IPv6 addresses.

- cons of User based rate limiting
  -  Rate limiting can be done only after user authentication (Once authenticated, the user will be provided with a token which the user will pass with each request). Therefore, rate limiting cannot be applied to login_in API itself.
