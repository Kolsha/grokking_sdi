## Instagram

**1. What is instagram ?**
- share photos and videos
- user can choose to share it publicly or privately
- instagram news feed

**2. Requirements and Goals of the System**
- **Functional Requirements**
  - user can share photos and videos;
  - user can follow other users;
  - the system should generate and display a user's news feed consisting of top photos from all the people the user follows;
  - user can perform search based on photo/video titles
- **Non-functional Requirements**
  - service should be highly available
  - for news feed generation, the latency <200ms
  - consistency can take a hit(i.e. suffer damage or loss) in the interest of availability (i.e. user does not see a photo for a while)
  - the system should be highly reliable (i.e. any uploaded photos/video should never be lost)

**3. Some Design Considerations**
- The system is a **read-heavy** system. Therefore, we will focus on building a system that can retrieve photos quickly.
  - **efficient management of storage** should be a critial factor in designing this system, becuase users can upload as many photos as they like;
  - low latency is expected while viewing the photos
  - **data should be 100% reliable** (i.e. if a user uploads a photo, the system will guarantee that it will never be lost)

**4. Capacity Estimation and Constraints**
- average photo file size ~= 200KB
- 2 Millions new photos every day
- **2 M * 200KB = 400GB per day**
- **400GB * 365 * 10 = 1460 TB for 10 years**

**5. High Level System Design**
