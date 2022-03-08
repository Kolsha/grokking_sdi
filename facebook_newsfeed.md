## Facebook Newsfeed

**1. What is facebook newsfeed ?**  
It includes status updates (with photos, videos, links, texts) and "likes" from **people, pages, and groups** that a user follows on facebook.
It is a compilation of your and your friends' life story (from photo, videos, locations).

**2. Requirements and Goals of the System**
- Functional Goals
  - generate newsfeed based on the posts from the people/pages/groups that a user follows
  - a user may follow many friends and a large number of pages/groups
  - feeds may contains videos/images/text
  - our service should support add new posts as they arrive to the newfeed for an **active** user. 
- Non-functional Goals
  - newsfeed generation latency ~2s
  - < 5s to add new arriving posts into a user's feed

**3. Capacity Estimation and Constraints**
- Traffic Estimates
  - 300M daily active users;
  - each user fetchs their timeline 5 times a day;
  - 300 M * 5 = 1.5 Billion timeline generations per day 
- Storage Estimates 
  - **Memory Estimates for cache**
    - 500 posts in every user's feed
    - each post ~1KB
    - need 500 * 1KB = **500KB per user **
    - need 500 KB * 300M = **150TB for all active users**(300M users)

**4. System APIs Design**
```
get_user_feed(api_dev_key, user_id, optional_since_id, optional_max_id, optional_count, optional_exclude_replies)
```
- Why need **api_dev_key** ?
  - The API developer key of a registered account. It can be used **to throttle users based on their allocated quota**.
- Why need **user_id** ?
  - generate the news feed for the user with this user_id.
- Why need **since_id** ?
  - get results more recent than the specified ID (i.e. since_id)
- Why need **count** ?
  - count = max num of feed items
- Why need **max_id** ?
  - get results older than the specified ID (i.e. the max_id)
- Why need **exclude_replies** ?
  - it's used to exclude replies (i.e. comments) from appearing in the returned new feed.
- **What's the result value ?**
  - a JSON object containing a list of feed items.

**5. Database Design(i.e. Design Data Schema)**
![facebook_newsfeed_database_schema](https://user-images.githubusercontent.com/26174882/157167064-4bbe14a6-0664-4025-94d5-ffe7dfa10c9b.jpeg)

**6. High Level System Design**
- **Feed Generation**
  - retrieve IDs of all users and entities that Jane follows;
  - retrieve the **latest, most popular and relevant posts** for those IDs.
  - rank those posts **based on the relevance to Jane**.
  - **store these posts in the cache and return the top posts(say 20) to be rendered on Jane's feed**.
  - when Jean reaches the end of her current feed, she can fetch the next 20 posts from **cache/server**.
  - **Note**:
    - Problem: you cannot always fetch posts from an non-empty cache, because the new incoming posts from people/entities that Jane follows, are not stored in cache.
    - How to fix this problem ? 
      - periodically (every 5 minutes) perform above steps. 
      - Jane can be **notified** in her feed that **she can fetch newer items**.

- **Feed Publishing**
  - user updates post or publish new posts
![facebook_newsfeed_highlevel](https://user-images.githubusercontent.com/26174882/157177502-26f7569b-995f-45c8-ac5f-55eeaa8ee34f.jpeg)


**7. Detailed Component Design**
- **How the Newsfeed Generation Service fetchs the most recent posts from all users and Janes follows ?**

```
(
# find posts from friend
SELECT FeedItemID FROM FeedItem WHERE UserId in (
  SELECT EntityOrFriendID FROM UserFollow WHERE UserId = JANE_ID and type = 0(user)
)

UNION

# find posts from pages/groups
SELECT FeedItemID FROM FeedItem WHERE UserId in (
  SELECT EntityOrFriendID FROM UserFollow WHERE UserId = JANE_ID and type = 1(entity)
)

# rank posts by creation date
ORDER BY CreationDate DESC

# return top 100 posts
LIMIT 100
)
```
  - **Problem**:
    - **slow** if Jane follows a lot of friends/pages/groups, because ranking on many many posts
    - 
