<p><a target="_blank" href="https://app.eraser.io/workspace/j6EpBKaYBeMiPq1umDCR" id="edit-in-eraser-github-link"><img alt="Edit in Eraser" src="https://firebasestorage.googleapis.com/v0/b/second-petal-295822.appspot.com/o/images%2Fgithub%2FOpen%20in%20Eraser.svg?alt=media&amp;token=968381c8-a7e7-472a-8ed6-4a6626da5501"></a></p>

# 02 - Scaling Database(s) Million(s)
#### Single Database Architecture
![image.png](/.eraser/j6EpBKaYBeMiPq1umDCR___JRTMbq9sLYe9p3tlCJ7gUsKxv173___image_vLjtK_3bLIN7wptJFXbXb.png "image.png")

- **Point in time recovery (PITR)**: It enables restoring a database to a specific, granular transaction moment by combining periodic full backups with continuous archiving of write-ahead logs (WAL) or binary logs (binlogs). Key components include immutable storage for backups, log streaming, and automation to replay logs to a desired timestamp. 
    - Costly service
    - Restores DB at the specified time, position
    - Life saver

- **Eventual consistency** is a consistency model in distributed systems where all data replicas are guaranteed to converge to the same value over time, provided no new updates are made to those items. It prioritizes system availability and low latency (AP in CAP theorem) over immediate, strict consistency, allowing temporary, inconsistent reads across different nodes.
#### Split Brain Problem
- **The Split Brain Problem**: The split-brain problem in system design is a situation in a distributed system, such as a cluster of servers, where the nodes become partitioned or disconnected from each other due to a network failure, but they all continue to operate independently. Each partition, or "brain," believes it is the sole active member of the cluster and starts making decisions or accepting writes.
- The critical issue arises when the network connection is restored. Since both partitions have been independently updating data, they now have conflicting states, leading to data inconsistency or corruption.
- To prevent split-brain, distributed systems use mechanisms like:
    - **Quorum:** Requiring a majority of nodes (a quorum) to agree before making a decision or performing a write operation. If a partition doesn't have a quorum, it shuts down or enters a read-only mode, preventing data conflicts.
    - **Fencing/STONITH (Shoot The Other Node In The Head)**: A mechanism used to ensure that a partitioned node is no longer modifying shared data or resources.

![image.png](/.eraser/j6EpBKaYBeMiPq1umDCR___JRTMbq9sLYe9p3tlCJ7gUsKxv173___image_0iUM3v4Tsn2jcOXpqBsmh.png "image.png")

![image.png](/.eraser/j6EpBKaYBeMiPq1umDCR___JRTMbq9sLYe9p3tlCJ7gUsKxv173___image_IYgVKpzwuT96Y77U0Nxd9.png "image.png")

- **Quorum / Majority Voting (>= 3votes)**: At least 2 nodes, should yes approval for write transaction, else the write operation will not take place. Another option is **ZooKeeper**. 
- For every database write operation, a lock has to be acquired for the operation to take place (like for example 30s). That’s why clicking the Buy **Now** button multiple times in a very short span of time is considered to be a single transaction only.
#### Distributed Locking
**1. The Two Purposes of Locks (Know Your Use Case)**

- **For Efficiency**: Used to avoid redundant work (e.g., sending the same email twice, double-processing a non-critical background job).
    - _Failure impact_: Minor cost increase or inconvenience.
    - _Recommendation_: A simple, single-node Redis lock is sufficient.

- **For Correctness**: Used to prevent concurrent processes from corrupting shared state (e.g., preventing lost updates in a database or file system).
    - _Failure impact_: Data corruption, permanent inconsistency, or severe system errors.
    - _Recommendation_: Requires rigorous consensus systems; do **not** use lightweight tools like Redis/Redlock for this.

**2. The Danger of Process Pauses & Network Delays**

- In distributed systems, a client holding a lock can experience unpredictable delays (e.g., Garbage Collection pauses, network packet drops, or CPU scheduler pauses).
- If a pause lasts longer than the lock's lease timeout (TTL), the lock expires and is granted to Client B.
- When Client A wakes up, it still _believes_ it holds the lock and proceeds to write to the shared resource, colliding with Client B and corrupting data.
- _Key Takeaway_: You cannot rely on a client "checking" its lock status right before a write, because a pause can happen immediately _after_ the check.
**3. The Solution: Fencing Tokens**

- To achieve true correctness, the lock service must issue a **Fencing Token** (a strictly monotonically increasing number) every time a lock is granted.
- The client must pass this token to the storage service with every write request.
- **Storage-Layer Enforcement**: The storage system must track the highest token it has seen. If it receives a write with an older (smaller) token, it must reject the request.
- _This guarantees safety even if the client is paused or delayed indefinitely._
- **GC**: Garbage Collector
![image.png](/.eraser/j6EpBKaYBeMiPq1umDCR___JRTMbq9sLYe9p3tlCJ7gUsKxv173___image_utHVRvg07CqHvj5XOzphE.png "image.png")

![image.png](/.eraser/j6EpBKaYBeMiPq1umDCR___JRTMbq9sLYe9p3tlCJ7gUsKxv173___image_UIGCt8ZwocmEl-uzZAFBv.png "image.png")

**4. Why Redis's "Redlock" Fails for Correctness**

- **No Fencing Tokens**: Redlock does not generate monotonically increasing tokens, making it fundamentally vulnerable to the process-pause race condition.
- **Dangerous Timing Assumptions**: Redlock assumes a _synchronous system model_. It relies heavily on wall-clock time (`gettimeofday` ) across multiple nodes.
- If a Redis node's clock jumps forward (due to NTP sync), or if network delays/client pauses exceed the lock TTL, the algorithm breaks and multiple clients can hold the lock simultaneously.
**5. System Design Recommendations**

- **When you only need efficiency**: Use a simple single-node Redis instance with asynchronous replication. Accept that it will occasionally fail.
- **When you need correctness**: Use a proper consensus system like **ZooKeeper** (which naturally provides `zxid`  or `znode`  version numbers as fencing tokens) or rely on a database with strong transactional guarantees (e.g., Postgres, Spanner).
- Always ensure the underlying storage resource actively participates in validating the lock via fencing tokens.
#### Single Region Critical Write
- Useful in Banking Systems (preferred)
- _Prefer Unavailability over Inconsistency_
- 




<!--- Eraser file: https://app.eraser.io/workspace/j6EpBKaYBeMiPq1umDCR --->