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
    - For live updates, the Newsfeed Generation Service needs to run every 5 minutes for every user. This results in high backlogs in this Newsfeed Generation Service.
    - For live updates, the Feed Notification Service needs to notify about newer posts to users every 5 minutes. This leads to very heavy loads.

  - **How to fix the problem above ?**
    - **Offline generation for newsfeed (i.e. pre-generation for newsfeed)**
      - **continuously** generate users's newsfeed and store them **in memory**.
      - therefore, using this scheme, user's newfeed is not compiled on load, but rather on a regular basis

    - **Pre-generate newfeed for all users ? That would consume too much memory. How to fix it ?**
      - remove user's newsfeed if this user has not accessed their newsfeed for a long time. How to know if a user does not log in for a long time ? **LRU cache**. 
      - Or figure out a login pattern of users: at what time of the day a user is active; which days of the week does a user access their newsfeed.
    - **How many feed items to store for every user ?**
      - if 1 page of a user's feed has 20 posts, and most users do not browse more than 10 pages of their newsfeed, we can decide to store only 200 posts per user.
      - **what if a user wants to see more posts** ? we can always query backend servers.
    - for every user, keep a struct object for him/her:
```
struct {
  LinkedHashMap<FeedItemID, FeedItem> feedItems;
  DateTime lastGenerated;
};
```

- **2 ways to publish a new post to all the followers: fanout-on-write(push approach) vs. fanout-on-load(pull approach)**
  - **fan-out-on-load (pull model)**
    - client issue a pull request.
    - **problem**:
      - clients need to issue a pull request so that they can see new data;
      - if there is no new data and client issues a pull request, this client is wasting resource
  - **fan-out-on-write (push model)**  
    - immediately push new feed to this feed publisher's followers.
    - users have to maintain a Long Poll request with the server in order to receive the updates.
    - **problem**: a user(celebrity) has millions of followers and this user sends a new post. The server has to push updates to millions of followers.
  - **hybrid**
    - stop pushing posts from celebrity users.
    - only push posts for those users who have a few hundred (or thousand) followers.
    - for celebrity users, the followers has to issue the pull request.
    - Or, once a user publishes a post, we limit the fanout to only her **online** friends.
  - **Should we always notify users if there are new posts available for their newsfeed ?**
    - may not for mobile devices, because data usage is relatively expensive and it can consume unnecessary bandwidth. Therefore, for mobile device, let users "Pull to Refresh" to get new posts.


**8. How to Rank Posts ?**
- by creation time; Or
- by importance
  - **How ?**:
    - select key "signals" that make post important and then find out how to combine them to calculate a final ranking score. 
    - signals include:
      - num of likes;
      - comments;
      - shares;
      - time of the update;
      - whether the post has images/videos 
