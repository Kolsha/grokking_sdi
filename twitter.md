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
    - 100 million tweets per day
    - 1 Tweet contains 140 characters
    - 1 character takes up 2 bytes
    - **Metadata** for each Tweet, such as ID, userID, timestamp (let's say 30 bytes)
    - 100 million * (280 + 30) bytes ~= 30GB per day
    - If every 5 tweet has a photo and every 10 tweet has a video, then the storage estimation would be
      - (100 million / 5) * 200KB per photo + (100 million / 10) * 2MB = 24TB per day
- **Bandwidth Estimates**
