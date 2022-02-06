## Typehead Suggestion

**1. Prerequisite**  
Leetcodes
- [Implement Trie](https://leetcode.com/problems/implement-trie-prefix-tree/)
- [Implement Trie II](https://leetcode.com/problems/implement-trie-ii-prefix-tree/)
- [Search Suggestions System](https://leetcode.com/problems/search-suggestions-system/)
- [Design Search Autocomplete System](https://leetcode.com/problems/design-search-autocomplete-system/)

**2. Benefits of Typehead ?**
- help users to articulate their search queries;
- guide users and lend them a helping hand in constructing their search query.

**3. Requirements and Goals of the Sytem**
- Functional Requirements: top 10 terms suggestions
- Non-functional Requirements: fast (i.e. real-time suggestions; low latency; within 200ms)

**4. Basic System Design and Algorithm**
- Trie;
- How to find top suggestions ?
    - sub-tree traversal using BFS/DFS (it is time-consuming the height of sub-tree is large); Or
    - **store top10 suggestions in each node** (it is storage-consuming);
       - we can optimize by storing only references of 10 terminal nodes;

- **How to update the trie ?**  
You cannot update the trie everytime the search engine receives a new query, because the search engine could receive 5 billion searches per day.
The solution is to **update trie offline after a certain interval**. Specifically,  
  - log queries and keep track of their frequencies;
  - set up [Map Reduce](map_reduce.md) to process the logging data periodically;
  - update trie with new data **offline. Do not block reading during updating. How ?**
      -  In each server, when updating the trie **offline**, make a copy of the trie then update this copy_trie. Once done, use the updated trie and discard the old one.
      -  For each trie server, we can have primary-secondary configuration for each trie server: update the secondary **while** the primary is serving traffice.

- **How to update frequencies of a phrase ?**
    - keep count of all the searched terms in the **last 10 days**.
    - and give **more weight to the latest searched** term.

- **Are there other ranking criterias for suggestions ?**
    -  freshness, user location, language, demographics, personal history, etc.

**4. How to store a Trie in a file ?**  
In case of a server goes down, you better store a Trie in a file in order to rebuild it when server restarts.
- with each node, we can store **(what character it contains)** + **(# of children it has)**;
- **right** after each node, we should **put all of its children**.

**5. Back-of-envelop estimation**
- **Storage Estimation**
    -  **How many unique terms needs to be stored per year ?** 5 billion queries. 20% of them are unique. And we only want to index(i.e. store) top 50% of these unique terms, because the rest 50% terms are less frequent. Therefore, the # unique terms needs to be stored per year = 5 billion * 20% * 50% = 0.5 billion
    -  assume every term consists of 3 words and every word consists of 5 characters. It takes 3 * 5 * 1 = 15 bytes for each term/query/phrase.
    -  In total, we need **15 bytes * 0.5 billion = 7.5 billion bytes = 7.5 GB**

**6. Data Partition: how we can efficiently partition our data and distribute it onto multiple servers ?**
- **Range Based Partitioning:**
    - all the terms starting with 'A' in one partition, all the terms starting with 'B' in one partition ....
    - **disadvantage**: lead to unbalanced servers
- **Partition based on the maximum capacity of the server**:
    - keep storing data on a server as long as it has memory available; Whenever a sub-tree cannot fit into a server, we _break our partition there_ to assign that range to this server and _move on to the next server_. Repeat this process.
    - _a load balancer is needed_ in front of the trie servers, in order to store mapping from what user typed to correct server.
    - Also, because we may get results from multiple servers, we need to merge the results. We may use an aggregator layer (to aggregate the results from multiple servers and return them to client).
    - **disadvantage**: still has hotspots issue.
- **Partition based on the hash of the term (Consistent Hashing)**:
    - by consistent hashing, our terms can be distributed randomly and hence minimize hotspots.
    - **disadvantage**: to find typeahead suggestions for a term, we have to ask **all servers** and then aggregate the results.

**7. Cache**  
- Keep a cache to **hold the most frequently searched terms and their typeahead suggestions**.
- Cache servers sits between application servers and trie servers.

**8. Replication and Load Balancer**
- Need replicas for trie servers.
- **Why need load balancer** ? Keeps track of our data partitioning scheme (i.e. when a request comes, load balancer needs to know which server this request is supposed to be routed to).

**9. What optimization can be done in client-side to improve user experience ?**
- only try to hit server(i.e. send request) if the user has not pressed any key for 50ms;
- client can _cancel the in-progress requests_, if user is constantly typing;
- client can _store the recent history of suggestions locally_;
- establish connection with the server _as soon as the user opens the search engine_;

**10. Other improvement**
- provide typeahead suggestions based on user's historical searches, location, languages

**11. Design Diagram**
![typeahead](https://user-images.githubusercontent.com/26174882/152669545-52007e40-679c-4154-abe8-09fee1b989a3.png)
