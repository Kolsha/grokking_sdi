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


**4. High-level System Design**  
- This system is a **read-heavy** system.
- We need a load balancer to distribute requests, therefore, we need multiple application servers.
- We need a database which can support a lot of reads.
- We need some file storage to store videos and photos.

![twitter_high_level](https://user-images.githubusercontent.com/26174882/156422106-9f3b70ab-8bea-461d-88c3-ffc73a4ff62e.png)


**5. Design Data Schema**  
Tweet Table
|   Primary Key |      tweet_id: int    |
| ------------- |  ------------- |
|   | content: varchar(140)|
|   | timestamp (i.e. createdTime): datetime|
|   | tweet_location: varchar(140)|
|   | user_location: varchar(140)|
|   | # of likes: int|

User Table
|   Primary Key |      user_id: int    |
| ------------- |  ------------- |
|   | name: varchar(140)|
|   | last_login_time: datetime|
|   | email: varchar(140)|
|   | account_creation_time: datetime|
|   | account_creation_time: datetime|

User Follows
|   Primary Key |      user_id: int    |
| ------------- |  ------------- |
|   | user_id_1: int|
|   | user_id_2: int|
|   | ...|

User Favorties
|   Primary Key |      user_id: int    |
| ------------- |  ------------- |
|   | tweet_id_1: int|
|   | tweet_id_2: int|
|   | ...|

We can store the above schema in a distributed key-value store to enjoy the benefits offered by NoSQL.

**6. Data Sharding**
- **Why do we need data sharding ?**
    - Since we have a huge number of new tweets per day and our read load is extremely high too, we need to distribute our data onto multiple machines such that we can read/write it efficiently.

- **How to shard our data ?**
    - sharding based on **user id**, using a hash function that will map the user to a database server.
        - **problem**: hot user; maintain a uniform distribution of data is difficult because some users can end up storing a lot of tweets and follows than other users.
    - sharding based on **tweet id**.
        - how to **search tweets(such as timeline generation)** in this case ?
            - _app server_ finds all users followed by this user;
            - _app server_ sends request to all database in order to find tweets published by these users;
            - every database finds tweets published by every user, and _sorts them by recency_ and returns top tweets (ranked by recency);
            - _app server_ merges all results together, _sort again_, and return top results to user.
        - **problem**: high latency
    - sharding based on **tweet creation time**
        - **advantage**: you can fetch all top tweets (ranked by recency) quickly
        - **problem**: traffic load is not distributed.
            - while reading, the server with latest data will have a very high load
            - while writing, all new tweets go to one server, and the remaining servers will be sitting idle. 
    - sharding **by tweet id AND tweet creation time**
        - **How ?**: use Tweet ID to reflect creation time.
        - **How to generate timeline in this case ?**
            - _app server_ finds all users followed by this user;
            - _app server_ sends request to all database in order to find tweet ids published by these users;
            - every database no need to read creation time of every tweet (**therefore reducing latency**). Instead, database can simply rank tweets by tweet id, because tweet id(i.e. the primary key) has epoch time included in ti.
            - _app server_ merges all results together, _sort again_, and return top results to user.
        - **advantage**:
            - one advantage is reducing latency, as you can see in above example
            - another one advantage is no need to write creationTime to database for every new tweet, therefore reducing write latency. 


