## TinyURL

**1. What is TinyURL ? What're the benefits of URL shortening ?**
- TinyURL(i.e. URL Shortening) is used to create shorter alias(i.e. short link) for a long URL.
- Benefits:
    - save a lot of space when displayed/printed/messaged/tweeted;
    - users are less likely to mistype shorter URLs.

**2. Requirements and Goals of the System**  
**Functional Requirements:**
- Given a URL, the tinyURL service generate a** shorter and unique **alias of it.
- User should optionally be able to **pick a custom short link (i.e. alias)** for their URL.
- The short url will **expire after a standard default timespan**. User should be able to **specify the expiration time**.

**Non-Functional Requirements:**
- highly available;
- URL redirection should happen in real-time with low latency;
- shortened links should not be guessable

**Other Requirements**
- **Analytics**, such as how many times a redirection happened ?
- TinyUrl service should also be accessible through **REST APIs**(_web service APIs/HTTP APIs, in which GET,POST,DELETE,PUT HTTP methods are used, adheres to REST architecture constraints. Therefore, web service APIs are called RESTful APIs_) by other services.
- Impose size limites on custom aliases to **ensure we have consistent URL database**. Let's assume user can specify a maximum of 16 chars per custom key.

**3. Back-of-envelop estimation**
- **Traffic Estimation**
    - 100 : 1 read/write ratio;
    - 50B per month: 500M per month;
    - **New URLs shortenings(i.e. write) per second**: 500 million / (30 days * 24 hours * 3600 seconds) = 192 URLs/s
    - **URL redirections**: 100 * 192 URLs/s = 19200 URLs/s
- **Storage Estimation**
    - 500M per month
    - assume we store every URL for 5 years
    - To be able to store URL for 5 years, we need to store 500M * 5 * 12 = 30 billion URLs;
    - Assume each stored object will be 500 bytes
    - We need storage: **30 billion * 500 bytes = 15TB **
- **Bandwith Estimation (incoming data to our service and outgoing data from our service)**
    - expect 192 URLs/s shortening(i.e. writing) request
    - **the incoming data = 192 * 500 bytes/s = 96KB/s**
    - expect 19200 URLs/s redirection(i.e. reading) request
    - **the outgoing data = 19200 * 500 bytes/s = 9.6MB/s**
- **Memory Estimation for Cache**
    - **80-20 rule**: 20% URLs generate 80% traffic. Therefore, we'd like to cache 20% hot URLs.
    - We want to cache every day's URLs.
    - Therefore, **19200 URLs/s * 24hours * 3600seconds * 20% * 500 bytes= 165GB**
    - The actual memory/cache cost is **< 165GB** because there are duplicate URLs reading requests.

**4. System Interface Design**
```
string createURL(api_dev_key, original_url, custom_alias=None, expiration_date=None, user_name=None)
# user_name: optional user name to be used in the encoding

# return value can be
# 1. shorten URL if successful Or
# 2. error code if failed
```

```
deleteURL(api_dev_key, short_url)

# return value can be
# 1. 'URL removed" if successful
# 2. error code
```
**The importance of using api_dev_key:** to make sure that malicious user cannot consume all URL keys in current design, by limiting # of URL shortening requests in certain period of time.

**5. Database Design**  
Because we anticipate storing billions of rows, we choose No-SQL database, such as DynamoDB, Cassandra.  
URL table
| PK  | hash_of_short_url: varchar |
| ------------- | ------------- |
|  | original_url: varchar  |
|  | dev_id/user_id: int  |
|  | creation_date: datetime  |
|  | expiration_date: datetime  |

User table
| PK  | user_id: int |
| ------------- | ------------- |
|  | last_login_date: datetime  |
|  | creation_date: datetime  |
|  | name: varchar  |
|  | email: varchar  |

**6. Basic System Design**  
- **some concepts to know**
    - base36 encoding: the hash value consists of characters from [a-z] + [0-9];
    - base62 encoding: the hash value consists of characters from [a-z] + [A-Z] + [0-9];
    - base64 encoding: the hash value consists of characters from [a-z] + [A-Z] + [0-9] + '+' + '/';
- **What's the length of the short key ?**
    - if the length of the short key = 6 and we use base64 encoding, it results in 64^6 = 68.7 billion possible strings. Therefore, let's say the **six letter key** would suffice our system.
- **How to generate a _short and unique key_ for a given URL ?**
    - **Option 1: encoding URL** 
        - **The problem is**: after md5 is applied to the original url, it will produce 128 bit hash value. Then after applying base64 encoding on this 12 bit hash value, it will produce a key with **length > 6**. However, the length of our key should be 6.
        - **How to solve this problem ?**  
        design diagram, regarding how to encode a long url:
        ![unique_url_encoding](https://user-images.githubusercontent.com/26174882/152709847-e0579e80-5952-4acd-9111-3b58b44e5988.png)
    - **Option 2: Generate keys offline**
         - have a standalone Key Generation Service(KGS) that generates random 6-letter strings beforehand and store them in database (let's call it **key-DB**).
         - therefore, we are not encoding the URL, so we do not need to worry about duplications or collisions.
         - KGS can use 2 tables to store keys: one for keys that are not used yet. one for all used keys.
         - But, **is there a concurrency problem ?**
            -  KGS must **synchronize(or get a lock on)** the data structure holding the keys **before** removing keys from it and giving them to a server.
         - To speed up, Application Server can cache some keys from key-DB.

**7. Data Partitioning and Replication**
- Range Based Partition
- Hash Based Partition using Consistent Hashing

**8. Cache **
- Because we need 165GB storage for cache, you can fit all the cache **into one machine**. Or we can **use a couple of smaller servers** to store all these hot URLs.
 
- design diagram with cache, regarding read request
![url_read_request](https://user-images.githubusercontent.com/26174882/152725204-91f95531-6fc5-494f-b4b0-ed326e3fa1be.png)

**9. Load Balancer**  
The load balancing layer can be added in 3 places:
- between client layer and application layer
- between application layer and database layer
- between application layer and cache layer

**10. Purging or DB clean up**  
lazy clean up:
- whenever a user tries to access an expired link, we can delete this link from database;
- run clean-up service, and run it **only when user traffic is expected to be low**.
- after removing a expired link, we can **put the key back to key-DB** to be reused.

**11. Telemetry**  
Store some statics such as
- user locations, such as the country of the visitor
- date and time of access
- web page that refers to the click
- platform from which the page is accessed

**12. Security and Permissions**  
Q: Can user create private URL which only allow a particular set of users to access ?  
A: Yes.
- set **permission level(private or public)** for each URL in the database;
- also **create a separate table** to store userIDs which have permission to a specific URL.

**13. High-level design diagram**
![tiny_url_design](https://user-images.githubusercontent.com/26174882/152726693-76afae16-129f-4f7d-9716-f616ba3c3b11.png)
