## THE PROBLEM

So we maintain an associative table that links users, labels, channels, and videos. This table acts as the central source of truth for access control across our platform. Because it’s queried for both search operations and permission checks, it experiences extremely high traffic.

As our user base and label count grew, the table became a major performance bottleneck. For users with thousands of labels, queries, especially those involving complex joins and large IN clauses, started taking up to 17 seconds at the p95. This led to a poor user experience and a spike in complaints.

To address this, I designed a multi-phase optimization strategy. The goal was to deliver incremental improvements, reduce query latency, and restore user trust as quickly as possible.

## THE SOLUTION

The first phase involved denormalizing our tables to consolidate all relevant data into a single structure. This eliminated the need for expensive joins during query execution. We also appleid a GIN index on the label array, which significantly optimized queries using large IN clauses, especially for users with access to thousands of labels. The impact was clear. we saw a 40% improvement in both p95 and p99 query latency. but even after these optimizations, video search times were still around 11 seconds, which was above our acceptable range. Our target was sub-3 second response times, so further optimization was necessary.

The second phase targeted the inefficiencies caused by querying large arrays of UUIDs. Remember we had a large IN-clause that is composed of large number of UUIDs. That was one of the root causes of the poor latency. UUIDs are excellent for uniqueness and security, but they’re not optimal for database performance, especially in large IN clauses. To address this, I introduced a mapping layer that converts UUIDs to integers. We created a dedicated table to store these mappings and leveraged caching to minimize lookup latency as well. I guess the drawback of introducing a new table and any associated mechanism means another maintenance point. For example if there is a bug in UUID to integer conversion logic, that exposes a risk of users being able to see other user's data. But this was necessary as INT type is so much faster when it comes to querying, we would just need to make sure that one UUID is only mapped to one Integer value. This was easily achieevable at DB level. Since we were using Relational DB, I used insert ONLY IF NOT EXIST at the transaction level. Otherwise, increase by 1 and re-insert.

By switching our queries to use integer IDs instead of UUID strings and storing the user to labels mapping in the cache, we dramatically reduced the computational overhead for the database. As a result, search times dropped from over 10 seconds to just a couple of seconds. This optimization was immediately noticeable to users. 

## THE ALTERNATIVES WE CONSIDERED

1. Sharding the Table

Split the large associative table into smaller shards based on user, label, or channel ranges.

Sharding the table can greatly improve query performance and scalability by splitting data into smaller, more manageable pieces. but, it introduces complexity to the next level in both application logic and database operations. You need to handle cross-shard queries, ensure data consistency, and manage rebalancing as data grows. Operational overhead increases, and debugging or maintaining the system can become more challenging. In short, sharding boosts performance but adds significant complexity to development and maintenance.

## LESSONS LEARNED

The biggest lesson here is that user experience has to come first. When search times hit 17 seconds, it wasn’t just a technical problem. It was a trust problem. Users were frustrated, and we needed to act fast. It’s important to communicate openly about what’s being fixed and set clear goals, like getting search under 3 seconds. Every technical decision, from denormalizing tables to switching data types, should be driven by the impact on users. In the end, speed matters, but earning and keeping our user trust matters even more.