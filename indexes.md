## Indexes

**1. Why need indexes ?**  
Searching a big database table is slow. Index is a data structure that directly points us to the location where actual data lives.

**2. How to create an index ?**  
- 1 example    
Index is mapping from a _search_key_ to _a pointer to a whole row in the database table_ (i.e. the search key is one of the column in the pointed row).

![WechatIMG61](https://user-images.githubusercontent.com/26174882/157592664-6dc35f3a-aacf-42f6-9e3f-a363172e74cc.jpeg)

- If a large data set is spread over several physical devices, we can build an index which maps from the search_key to the physical device.

**3. How do indexes decrease write performance ?**
- Every time adding rows or making updates to existing rows **for a table with an active index**, not only we have to write the data to the database but **also have to update the index**.
- If your database is designed to be written often and rarely read from, using indexes is probably not worthy.
