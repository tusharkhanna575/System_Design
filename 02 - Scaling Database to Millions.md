<p><a target="_blank" href="https://app.eraser.io/workspace/j6EpBKaYBeMiPq1umDCR" id="edit-in-eraser-github-link"><img alt="Edit in Eraser" src="https://firebasestorage.googleapis.com/v0/b/second-petal-295822.appspot.com/o/images%2Fgithub%2FOpen%20in%20Eraser.svg?alt=media&amp;token=968381c8-a7e7-472a-8ed6-4a6626da5501"></a></p>

# 02 - Scaling Database(s) Million(s)
#### Single Database Architecture
![image.png](/.eraser/j6EpBKaYBeMiPq1umDCR___JRTMbq9sLYe9p3tlCJ7gUsKxv173___image_rwOFNK59fZ-vEflbW0opL.png "image.png")

- **Point in time recovery (PITR): **It enables restoring a database to a specific, granular transaction moment by combining periodic full backups with continuous archiving of write-ahead logs (WAL) or binary logs (binlogs). Key components include immutable storage for backups, log streaming, and automation to replay logs to a desired timestamp. 
    - Costly service
    - Restores DB at the specified time, position
    - Life saver

- **Eventual consistency **is a consistency model in distributed systems where all data replicas are guaranteed to converge to the same value over time, provided no new updates are made to those items. It prioritizes system availability and low latency (AP in CAP theorem) over immediate, strict consistency, allowing temporary, inconsistent reads across different nodes.
#### Split Brain Problem
- **The Split Brain Problem: **The split-brain problem in system design is a situation in a distributed system, such as a cluster of servers, where the nodes become partitioned or disconnected from each other due to a network failure, but they all continue to operate independently. Each partition, or "brain," believes it is the sole active member of the cluster and starts making decisions or accepting writes.
- The critical issue arises when the network connection is restored. Since both partitions have been independently updating data, they now have conflicting states, leading to data inconsistency or corruption.
- To prevent split-brain, distributed systems use mechanisms like:
    - **Quorum:** Requiring a majority of nodes (a quorum) to agree before making a decision or performing a write operation. If a partition doesn't have a quorum, it shuts down or enters a read-only mode, preventing data conflicts.
    - **Fencing/STONITH (Shoot The Other Node In The Head):** A mechanism used to ensure that a partitioned node is no longer modifying shared data or resources.









<!--- Eraser file: https://app.eraser.io/workspace/j6EpBKaYBeMiPq1umDCR --->