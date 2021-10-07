## 设计一个TinyUrl

为什么需要短域名服务呢?

1. URL shortening is used to create shorter aliases for long URLs.

Short links save a lot of space when displayed, printed, messaged, or tweeted. Additionally, users are less likely to mistype shorter URLs.

好看,节约空间.

**You should always clarify requirements at the beginning of the interview. Be sure to ask questions to find the exact scope of the system that the interviewer has in mind.**



**Functional Requirements:**

1. Given a URL, our service should generate a shorter and unique alias of it. This is called a short link.

2. When users access a short link, our service should redirect them to the original link.

3. Users should optionally be able to pick a custom short link for their URL.

4. Links will expire after a standard default timespan. Users should be able to specify the

   expiration time.

**Non-Functional Requirements:**

1. The system should be highly available. This is required because, if our service is down, all the URL redirections will start failing.
2. URL redirection should happen in real-time with minimal latency.
3. Shortened links should not be guessable (not predictable).

**Extended Requirements:**

1. Analytics; e.g., how many times a redirection happened?
2. Our service should also be accessible through REST APIs by other services.

**3. Capacity Estimation and Constraints**

Our system will be read-heavy. 读压力比较大.There will be lots of redirection requests compared to new URL shortenings. Let’s assume 100:1 ratio between read and write.

**Traffic estimates**

Assuming, we will have 500M new URL shortenings per month, with 100:1 read/write ratio, we can expect 50B redirections during the same period:

What would be Queries Per Second (QPS) for our system? New URLs shortenings per second:

500 million / (30 days * 24 hours * 3600 seconds) = ~200 URLs/s

Considering 100:1 read/write ratio, URLs redirections per second will be:

100 * 200 URLs/s = 20K/s

**Storage estimates**

 Let’s assume we store every URL shortening request (and associated shortened link) for 5 years. 