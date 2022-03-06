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
