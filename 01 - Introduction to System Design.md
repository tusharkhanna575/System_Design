---
title: Introduction to System Design
---

<p><a target="_blank" href="https://app.eraser.io/workspace/zlLTDxiAatIMepkFzrk1" id="edit-in-eraser-github-link"><img alt="Edit in Eraser" src="https://firebasestorage.googleapis.com/v0/b/second-petal-295822.appspot.com/o/images%2Fgithub%2FOpen%20in%20Eraser.svg?alt=media&amp;token=968381c8-a7e7-472a-8ed6-4a6626da5501"></a></p>

# 01 - Introduction to System Design
>
> _**"Never start designing, ask questions first.”**_

- Start with the design of a system which should be able to handle at least 1 user.
- **1 -> 10K -> 1M**
![image.png](.eraser/zlLTDxiAatIMepkFzrk1___JRTMbq9sLYe9p3tlCJ7gUsKxv173___image_hA2MSIPd8-vb5Zl7UD6LW.png "image.png")

- **DNS**: Domain Name Server
- The process is called **domain name resolution**
- ([google.com](http://google.com/)  -> IP Address)
- The protocol followed is **ARP** (Address Resolution Protocol)
- **Bottleneck** can be useful as well as not useful
- System Design is all about **Tradeoffs** (in the form of storage, CPU, RAM, etc.)
- **Vertical Scaling / Scaling Up**: increase system capacity by adding more resources - CPU, RAM, or storage to a single server.
- **SPOF (Single Point of Failure)**
- Keep backup server(s)
- **CAP Theorem:**
  - **Consistency**
  - **Availability**
  - **Partition - Tolerance**

- **Stateful Servers**: the server keeps track of the state of each session or interaction and maintains that information based on the user's past requests. **Vertical Scaling** is difficult (downtime, SPOF, high RAM consumption, resource intensive)
- **Web socket** servers are stateful servers, hence hard to scale
- Discourage use of stateful servers
- **Stateless servers** are easy to scale, because connection can be made to any server, reducing SPOF, increasing availability, solves money problem by auto-scaling
- **Horizontal Scaling / Scaling out**: increases capacity by adding more machines to a system, improving fault tolerance and handling high, fluctuating traffic via load balancing.

![image.png](.eraser/zlLTDxiAatIMepkFzrk1___JRTMbq9sLYe9p3tlCJ7gUsKxv173___image_L5KnVCm9qp1anjeI4Q-Lj.png "image.png")

- **Vertical scaling / Scaling up**: enhances system capacity by upgrading existing hardware—adding CPU, RAM, or storage—to a single server, making it more powerful rather than adding more servers. It is best suited for monolithic applications or databases needing a simple, fast performance boost without complex, distributed architecture, but is limited by the maximum capacity of one machine.
![image.png](.eraser/zlLTDxiAatIMepkFzrk1___JRTMbq9sLYe9p3tlCJ7gUsKxv173___image_XT0HxRX_Uh-SZpDrNZbp4.png "image.png")

- **Load Balancer:** balances equal load between all the server(s), by serving as a **reverse-proxy** between user & server, also going to server for the request’s response on user’s behalf.
- In DNS, we keep the IP Address of load balancer
- Servers will be having private IP (so as to protect them from direct access to user’s), load balancer is kept in the same network as server.
- In AWS, **Virtual Private Cloud (VPC)** is same as load balancer
- In the worst case, load balance is SPOF (very rare scenario).
- Atlassian load balancer failure case study [Atlassian Load Balancer Case Study](https://medium.com/@codesprintpro/lessons-from-the-atlassian-outage-understanding-load-balancers-and-blue-green-deployment-4fb2778eab0e)
- Each platform /application will have its own system design strategy
- **Pre - Warming:** proactively initializing resources or loading data before they are requested to eliminate latency.
- _For example:_ Youtube, Netflix, Hotstar all are video streaming platforms but have different system design strategy.
![image.png](.eraser/zlLTDxiAatIMepkFzrk1___JRTMbq9sLYe9p3tlCJ7gUsKxv173___image_pvjAK5-zIkxNesk1YXIxt.png "image.png")

- **TTL**: time to live (amount of time after which cache to be cleared)
- A **spike** in system design is a time-boxed, Agile technique used to research, prototype, or investigate technical uncertainties, risks, or complex requirements before full-scale development
![image.png](.eraser/zlLTDxiAatIMepkFzrk1___JRTMbq9sLYe9p3tlCJ7gUsKxv173___image_W74g7wjMo9ZvS8CMvEPBs.png "image.png")

- There may be some cases like cache that expired or cleared, at that moment only the database load suddenly spikes up, leading the system(s) to crash.
- **Cache invalidation** is the critical process of removing or updating stale data from a system's cache when the underlying source of truth changes. It ensures data consistency and prevents users from seeing outdated information.
- In order to handle this, we use some **cache expiry techniques**, namely:
  - **Jitter on TTL**:
    - Breaks the synchronization of cache expiry events by adding a small, random time offset to the set Time-To-Live (TTL).
    - Prevents the "Thundering Herd Problem" by ensuring all keys do not expire at the exact same moment.
    - Distributes the load of cache recomputation over a small time window, protecting the backend.

  - **Probability Early Computation**:
    - The cache entry is recomputed and refreshed before its actual TTL expires.
    - The probability of initiating a recompute increases as the entry's age approaches its expiration time.
    - Used for high-traffic, time-sensitive events like:
            1. IRCTC (tatkal booking)
            2. Flipkart Big Billion Days (12 AM)
            3. Zomato (lunch time on weekdays, 12 - 2 PM)

  - **Mutex / Lock (Binary Semaphore)**:
    - Ensures only a single request performs the expensive cache recomputation when a cache miss occurs.
    - The first request acquires a binary lock on the key, and subsequent requests wait for the result or are served a stale value.
    - Effectively prevents the Thundering Herd from hitting the backend, as the work is strictly serialized.

  - **Stale while Revalidating (SWR)**:
    - Serves the currently cached, but expired ("stale"), value immediately to the user to achieve low latency.
    - Triggers the request for new data (revalidation) in the background, minimizing the user's wait time.
    - Used in applications like Hotstar and leaderboard designs where slight data lag is acceptable in exchange for speed.

  - **Cache Warming (Pre - warming)**:
    - Involves proactively loading data into the cache before it is requested by users.
    - Used specifically to prevent the "cold-start penalty" right before a known, scheduled traffic spike.
    - Applied in scenarios such as BookMyShow on Friday (9 AM) when ticket sales are expected to surge.Used in BookMyShow Friday (9 AM)

- The above scenario(s) is commonly known as **The Thundering Herd Problem**.
- Artifacts [https://app.eraser.io/workspace/SVBgeRmqA2BgKoaeHhct](https://app.eraser.io/workspace/SVBgeRmqA2BgKoaeHhct)

<!--- Eraser file: https://app.eraser.io/workspace/zlLTDxiAatIMepkFzrk1 --->