## Twitter Search

**1. What is Twitter Search ?**  
A system that allows searching over all the user tweets.

**2. Requirements and Goals of the System**
- a system that can **efficiently store and query** tweets.

**3. Capacity Estimation and Constraints**
- we have 400 millions tweets every day; say each tweet is 300 bytes, the total storage is 400million * 300 bytes = **120GB per day** (storage estimation);
- we will have _500 millions searches_ every day.
- the search query will **consist of multiple words with AND/OR**.
- Twitter has 1.5 billion users with **800 million daily active users**.

**4. System APIs**

```
search(api_dev_key, search_terms, maximum_results_to_return, optional_sort, page_token)
```
optional_sort (int):
- 0: latest_first
- 1: best matched
- 2: most liked

returns:
- a **JSON** object containing information about a list of tweets matching the search query(i.e. the search terms).
- each result entry can have the user id & user name, tweet text, tweet id, creation time, num of likes, etc.

**6.Detailed Component Design**
- **Data Partition/Sharding**
  - 120GB per day * 365 days per year * 5 years = 219TB to store tweets for the next 5 years.
  - If **we never want to be more than 80% full at any time**, we need 219TB / 0.8 = 270TB of total storage. 
  - If we want to **keep an extra copy of all tweets for fault tolerance** (i.e. data replication), we need 270TB * 2 = 540TB of total storage.
  - Therefore, we need 540TB / 4TB per server = 135 servers to hold all the tweets.
  - We can **partition our data based on TweetID**.

- **Index**
  - **What should our index look like ?**
    - a big **distributed** hash table
    - _key_ is the _word_
      -  all English words ~ 300K
      -  some famous nouns such as people names, city names... 200K
      -  we need 500K * (5 characters on average for each word) = 2.5MB for the keys.
    - _value_ is _a list of TweetIDs_ of all the tweets which contain that word
      - ~300 Billion tweets in 2 years (**assume that we keep the index in memory for all the tweets from only past 2 years**)
      - we need 300 billion * (5 bytes for each tweet id) = 1500 GB for the values.
    - let's assume that on average we have 15 words in each tweet (40 words in each tweet - prepositions and other small words like 'the', 'an', 'and' etc)
    - therefore, each tweet id will be stored 15 times.
    - therefore, to store the index (i.e. hash table), the storage is 1500 GB * 15 + 2.5MB ~ = 22.5TB
