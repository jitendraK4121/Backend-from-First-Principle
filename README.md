# Backend-from-First-Principle
learning backend


## Caching from First Principles: The Ultimate Guide for Backend Developers 🚀

If you've ever been amazed by the speed of Google Search, the seamless streaming of Netflix, or the instant updates on X (Twitter), you've experienced the power of a fundamental engineering concept: **caching**. For backend developers, understanding caching isn't just a "nice-to-have"—it's an absolute necessity for building fast, scalable, and resilient applications.

This guide will break down caching from the ground up, covering everything from high-level concepts to the practical mechanics you'll use in your day-to-day work, leaving no stone unturned.

### **What Exactly Is Caching?**

Let's start with two definitions from the video, from simple to technical.

> In one sentence: **Caching is a mechanism using which we decrease the amount of time and effort it takes to perform some amount of work.**

This "work" could be a complex database query, a heavy computation, or transferring a large file. Caching makes this work "cheaper" by storing the result so we don't have to do it over and over again.

> A more technical definition: **Caching is the practice of keeping a subset of data in a location that is faster to access than its primary storage.**

We don't cache everything, just a carefully selected **subset** of data. This selection is based on factors like frequency of use and the probability of it being needed again soon. The goal is to improve performance in high-stakes applications where latency is measured in milliseconds or even **two-digit microseconds**.

-----

### **Why Caching is Non-Negotiable: Lessons from Tech Giants**

The importance of caching becomes crystal clear when we look at how the biggest names in tech rely on it.

#### **1. Google Search: Taming Billions of Web Pages** 🔍

Every time you hit "Enter" on a Google search, you're kicking off a process that would otherwise be incredibly slow.

  * **The Challenge (Without Caching):** For a common query like "what is the weather today," Google's search engine would have to execute its complex workflow—**crawling, indexing, and ranking billions of web pages**. This is **computationally expensive**, requiring massive CPU and memory resources. If this happened for every single one of the millions of times this query is searched daily, the latency would be unbearable and the server load unsustainable.

  * **The Solution (With Caching):** Google uses a massive, **distributed in-memory caching system**.

    *(Below is the Mermaid code for the flowchart. You can paste this into a compatible editor like Notion, GitHub, or an online Mermaid editor to generate the visual diagram.)*

    ```mermaid
    graph TD
        A[User searches "what is weather today"] --> B{Check Cache for Query};
        B -- Cache Hit (Found) --> C[✅ Return Cached Result Instantly];
        B -- Cache Miss (Not Found) --> D[🔥 Perform Expensive Search Operation];
        D --> E[Return Result to User];
        E --> F[💾 Store Result in Cache];
    ```

    When the system finds the data in the cache, it's called a **cache hit**. When it doesn't, it's a **cache miss**. By caching the results of common queries, Google ensures near-instantaneous responses for the vast majority of users.

#### **2. Netflix: Delivering Terabytes to Your Screen** 🎬

Netflix streams enormous amounts of data (movies, animes, series, etc.) to millions of users worldwide. A single movie file is often stored in multiple resolutions (e.g., 480p, 720p, 1080p) through a process called **encoding**. This allows the platform to dynamically send an optimized version based on your device and network speed, saving your bandwidth and reducing their server load.

  * **The Challenge (Without Caching):** If every user streamed directly from Netflix's main "originating servers" (let's say, in the US), a user in India would experience significant buffering and lag due to the vast physical distance the data must travel. The originating servers would be overwhelmed.

  * **The Solution (With Caching):** Netflix uses a **Content Delivery Network (CDN)**. This is a network of servers, called **edge locations**, strategically placed all over the world. These edge servers cache a subset of Netflix's content—specifically, the movies and shows that are popular in that geographic region, based on trend analysis and machine learning. When you press play, you're connecting to the nearest edge server in your region, which delivers the cached video file with minimal latency. Other platforms like **Vercel** use similar **edge networks** to serve static web assets (HTML, CSS, JavaScript) quickly.

#### **3. X (Twitter): Capturing Real-Time Trends** 🔥

Identifying trending topics on X involves analyzing millions of tweets in real-time with expensive machine learning algorithms running on powerful infrastructure (like GPUs).

  * **The Challenge (Without Caching):** If X re-calculated the trending list every time a user visited the trending section, the servers would crash in minutes under the load of millions of concurrent requests.

  * **The Solution (With Caching):** X pre-computes the trending topics for different regions **every few minutes**. These results are then stored in a fast in-memory cache like **Redis**. When you open the app, you're simply fetching this pre-computed list from the cache. A trend like an ongoing election won't change in seconds, so it's safe to cache for a few minutes or hours.

**The Common Pattern:** As you can see, caching is the go-to solution in two primary scenarios:

1.  When you want to avoid repeating **heavy computation**.
2.  When you want to avoid transferring **large amounts of data** over long distances.

-----

### **The Caching Hierarchy: A Backend Developer's Map**

As a backend engineer, you'll encounter caching at three main levels.

### \#\# **Level 1: Network-Level Caching**

This type of caching focuses on reducing network latency by moving data closer to the user.

#### **Content Delivery Networks (CDNs) In-Depth**

CDNs cache static content like videos, images, and web pages.

*(Below is the Mermaid code for the flowchart describing the CDN process.)*

```mermaid
graph LR
    subgraph User
        A[User Enters URL]
    end
    subgraph Internet
        B[1. DNS Query]
        C[2. CDN DNS Routes to Nearest POP]
        D[3. Request Hits Edge Server]
    end
    subgraph EdgeServer [Edge Server in Point of Presence (POP)]
        E{Check Local Cache}
    end
    subgraph OriginServer [Origin Server]
        H[Primary Data Store]
    end

    A --> B --> C --> D --> E;
    E -- Cache Hit --> F[✅ 4a. Serve Content to User];
    E -- Cache Miss --> G[🔥 4b. Fetch from Origin Server];
    G --> H;
    H --> G;
    G --> I[💾 5. Store in Cache (with TTL)];
    I --> J[✅ 6. Serve Content to User];
```

A crucial concept here is **TTL (Time to Live)**. This is a configuration that tells the CDN how long to keep an item in its cache. Once the TTL expires, the CDN will fetch a fresh copy from the originating server on the next request, ensuring content doesn't become stale.

#### **DNS Caching In-Depth**

The Domain Name System (DNS) is the phonebook of the internet. It translates domain names (`google.com`) into IP addresses (`142.251.42.78`). This lookup process is called resolution, and it's heavily cached to avoid delays.

*(Below is the Mermaid code for the flowchart describing the layered DNS caching process.)*

```mermaid
graph TD
    A[User types example.com] --> B{Browser Cache?};
    B -- Yes --> Z[✅ Found!];
    B -- No --> C{OS Cache?};
    C -- Yes --> Z;
    C -- No --> D[Query Recursive Resolver (ISP/Google)];
    D --> E{Resolver Cache?};
    E -- Yes --> Z;
    E -- No --> F[🔥 Start Recursive Lookup];
    F --> G[1. Query Root Server];
    G --> H[2. Query TLD Server (.com)];
    H --> I[3. Query Authoritative Server (for example.com)];
    I --> J[IP Address Found!];
    J --> D;
    D --> K[💾 Cache the Result];
    K --> Z;
```

The query first checks the browser cache, then the OS cache, then your ISP's resolver cache. Only if it's not found in any of these caches does the full, multi-step lookup to the root servers occur.

-----

### \#\# **Level 2: Hardware-Level Caching & RAM**

At the lowest level, your computer's hardware has its own hierarchy of caches designed for maximum speed, as shown in the diagram below.

As the diagram illustrates, data access speed forms a pyramid.

  * The **CPU** is at the top. To avoid waiting for data, it has its own small, ultra-fast caches.
  * **L1 and L2 Caches** are small caches dedicated to a single CPU core.
  * **L3 Cache** is a larger cache shared by multiple CPU cores.
  * **RAM (Main Memory)** is the next level down. It's slower than the CPU caches but much faster than the disk.
  * **Hard Disk / SSD** is the primary, persistent storage. It has the largest capacity but is also the slowest.

For backend developers, the most important layer to understand is **RAM (Random Access Memory)**.

  * **Why is RAM so fast?** Accessing data in RAM is an electronic operation with no moving parts. It can access any memory address directly in roughly the same amount of time, hence "Random Access." This is orders of magnitude faster than a hard disk, which has a mechanical head that must physically move to read data.
  * **The Trade-Offs:**
      * **Speed vs. Cost & Capacity:** RAM is significantly more expensive and has a much smaller capacity than disk storage.
      * **Speed vs. Persistence:** RAM is **volatile**. When the power goes off, all data stored in it is lost. Disk storage is **non-volatile** or **persistent**—it retains data without power.

This speed-persistence trade-off is precisely why **in-memory databases** exist.

-----

### \#\# **Level 3: Software-Based Caching (In-Memory Databases)**

This is where you, as a backend developer, will spend most of your time. We call it "software-based" because we interact with it through libraries and APIs, but its performance comes directly from leveraging the hardware-level speed of RAM.

Popular technologies include **Redis**, **Memcached**, and cloud-based services like **AWS ElastiCache**. These are typically categorized as **in-memory, key-value, NoSQL databases**.

  * **In-Memory:** They store data primarily in RAM for maximum speed. They often use secondary storage (disks) for persistence, like saving a snapshot of the data to disk periodically so it can be reloaded after a restart.
  * **Key-Value:** The data model is beautifully simple. You store a value (like a string, a list, or a JSON object) and associate it with a unique key. To retrieve the value, you just provide the key. It's like a super-fast dictionary.
  * **NoSQL:** They don't enforce the rigid schemas (tables, rows, columns) of traditional SQL databases, offering more flexibility.

-----

### **Mastering In-Memory Caching: Strategies and Policies**

When using a cache like Redis, you need to make two key decisions: how to put data in, and how to take data out when it's full.

#### **Caching Strategies (How data gets IN)**

1.  **Lazy Caching (or Cache-Aside):** This is the most common and intuitive strategy.

    1.  The application first requests data from the cache.
    2.  If the data is **not** in the cache (a cache miss), the application requests the data from the primary database.
    3.  The application then stores the retrieved data in the cache.
    4.  Finally, the application returns the data to the client.
        The application is responsible for "lazily" populating the cache only when data is actually requested and not found.

2.  **Write-Through Caching:** In this strategy, the cache is updated at the same time as the database during a write operation (e.g., a `POST` or `PUT` request).

    1.  The application writes the new/updated data to the cache.
    2.  The application then writes the same data to the primary database.

    <!-- end list -->

      * **Advantage:** The cache is always fresh and consistent with the database.
      * **Disadvantage:** It adds latency (overhead) to write operations because you're writing to two systems.

#### **Cache Eviction Policies (How data gets OUT)**

Because cache memory is limited, you need an **eviction policy** to decide which items to remove when the cache is full.

  * **No Eviction:** The cache will simply return an error when you try to add an item to a full cache. (Rarely used).
  * **LRU (Least Recently Used):** Removes the item that has the oldest access timestamp. If you have four items and one was last accessed yesterday while the others were accessed today, the one from yesterday gets evicted.
  * **LFU (Least Frequently Used):** Removes the item that has been accessed the fewest number of times. This is useful for keeping popular items in the cache even if they aren't accessed very recently.
  * **TTL (Time To Live):** Items can be set with an expiration duration. Redis can automatically remove keys that have expired, which is a form of eviction.

-----

### **Real-World Backend Use Cases for Redis**

Here are four common patterns where you'll use an in-memory cache like Redis in your backend applications.

1.  **Database Query Caching:**

      * **Problem:** You have a slow SQL query with many `JOIN`s and aggregations that runs on a frequently visited page (like a dashboard or product page). This puts a heavy load on your database.
      * **Solution:** Execute the query once, then cache the result in Redis with a TTL (e.g., 1 hour). Subsequent requests will hit the cache, protecting your database. This is ideal for **read-heavy, write-infrequent** data like e-commerce product details or social media user profiles.

2.  **Session Storage:**

      * **Problem:** In a typical authentication flow, you generate a session token for a user. On every subsequent request, you need to validate this token. Checking it against your primary database every time adds latency and load.
      * **Solution:** Store the session tokens in Redis. Validating a session becomes a lightning-fast key lookup in Redis, offloading your main database and speeding up every single authenticated API call.

3.  **API Caching:**

      * **Problem:** Your service relies on a third-party API (e.g., a weather API) that has usage-based billing or strict rate limits. Calling it on every user request can get expensive or get you blocked.
      * **Solution:** Call the external API once, cache the response in Redis with a reasonable TTL (e.g., 5 minutes for weather data), and serve all subsequent requests from your cache until the data expires.

4.  **Rate Limiting:**

      * **Problem:** You need to prevent abuse by limiting the number of requests a user can make to an API endpoint in a given time frame (e.g., 50 requests per minute).
      * **Solution:** Redis is perfect for this. When a request comes in, a middleware can:
        1.  Get the user's IP address (often from the `X-Forwarded-For` request header).
        2.  Use the IP as a key in Redis and a counter as the value.
        3.  Set an expiry on the key (e.g., 1 minute).
        4.  Increment the counter on each request. If the count exceeds the limit (50), reject the request with a **`429 Too Many Requests`** status code.
      * This is far more efficient than using a relational database, as it avoids database load and keeps API latency minimal.

### **Conclusion: Time to Get Your Hands Dirty**

We've covered a lot, but the core ideas are straightforward. Caching is about making intelligent trade-offs—trading limited, volatile memory for incredible speed. It's a fundamental technique for reducing latency, decreasing server load, and creating a better user experience.

The best way to solidify this knowledge is to apply it. Pick an open-source tool like **Redis**, install a client library for your favorite programming language (like **`node-redis`** for Node.js), and start experimenting. Try caching a database query and measure the difference in response time. You'll see the power of caching firsthand.
