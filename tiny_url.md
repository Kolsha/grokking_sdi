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
    - **The problem is**: after md5 is applied to the original url, it will produce 128 bit hash value. Then after applying base64 encoding on this 12 bit hash value, it will produce a key with **length > 6**. However, the length of our key should be 6.
    - **How to solve this problem ?**
  ![unique_url_encoding](https://user-images.githubusercontent.com/26174882/152709847-e0579e80-5952-4acd-9111-3b58b44e5988.png)
