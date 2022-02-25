## Twitter

**1. Requirements**  

**Funtional Requirements**
- post tweet; follow other users;favorite tweets; display a user's timeline
- tweet can contain photos and videos

**Non-functional Requirements**
- service is highly available
- timeline generation is within 200ms
- consistency can take a hit in the interest of availability (i.e. sacrifice consistency for availability)

**Extended Requirements**
- Tweet search;
- reply to tweet
- moments
- trending topics
- tagging other users
- Who to follow ? (Suggestions)

**2. Back-of-envelop Estimation**  
- 200 million **DAU (Daily Active User)**
- **How many tweet views per day ?**
    - 200 million DAU + (2 views of his own timeline + 5 views of other user's timeline) * 20 tweets per timeline = 28 Billion tweets view per day 
- **Storage Estimation**
    - 100 million _new_ tweets per day
    - 1 Tweet contains 140 characters
    - 1 character takes up 2 bytes
    - **Metadata** for each Tweet, such as ID, userID, timestamp (let's say 30 bytes)
    - 100 million * (280 + 30) bytes ~= 30GB per day
    - If every 5 tweet has a photo and every 10 tweet has a video, then the storage estimation would be
      - (100 million / 5) * 200KB per photo + (100 million / 10) * 2MB = 24TB per day
- **Bandwidth Estimates**
    - incoming data(**ingress**) to our service is 24TB per day  = 290MB/sec
    - outcoming data(**outgress**)
        - 28 billion tweets * (280 bytes for text tweet) / (86400s per day) +
        - (28 billion tweets / 5) * 200KB / (86400s per day) +
        - (28 billion tweets / 10 / 3) * 2MB / (86400s per day) 
            - 3 represents that user _watch_ every 3rd video they _see_ in the timeline.

**3. System API**

```
string tweet(account_id, tweet_data, tweet_location, user_location, media_ids)
```

**Note:**  
1. media_ids is the id of media photo, video which needs to be uploaded separately;
2. **return value**: the URL to access the tweet if success. An error is returned if failure.
