In-memory database, used in front of a DB to fetch frequently used data.

![[Pasted image 20260515223547.png]]

![[Pasted image 20260515223640.png]]

**Key value pair**: Data stored like in hashmap.
**TTL (Time to live)** : Data expires after a certain period of time.

### Used in :
Rate limiting
Job queue
Background Workers
OTP Store
Session store
Cache data

### Used when :
Need to reduce read pressure
Temp data expiry with time
Shared counters like like,share etc

### Similar to Redis:
(Drop and replacement) - (Replacing just API keys of redis from these DBs works)
DragonflyDB
KeyDB
Valkey
Memcached
Upstash