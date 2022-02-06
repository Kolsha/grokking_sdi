| Master Database  | Slave Database |
| ------------- | ------------- |
| only supports write operations  | only supports read operations |
| it's updated by user's write operations such as insert, delete, update  | it's updated by syncing up with master database periodically |
| # of master database is small compared with # of slave database | # of slave database is large because of higher ratio of reads to writes |
| if all master databases go offline, a slave database will be promoted to be the new master database. This process is often complicated. | when 1 slave database goes offline, read operations are redirected to other healthy slave database; If all slave databases go offline, read operations will be redirected to the master database temporarily. |


## Advantages of Master-Slave Model used in Data Replication
- improve efficiency (i.e. performance) because it allows more queries to be processed in parallel.
- higher reliability and availability;
