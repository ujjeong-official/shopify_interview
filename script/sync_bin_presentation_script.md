I'm going to briefly walk you through how the Video Claiming architecture looks like at a very high level view. I'm not gonna go into details of how I come up with each components by doing this, this is just to give you a context of how things work in the Video Claiming platform. And then I can do the deep-dives on some of the challenges I encountered while building the product.




## THE PROBLEM

Our platform manages hundreds of thousands of YouTube videos and channels. We need to keep them synchronized with YouTube's API - we would need to automatically check for fresh claim data, if there are any metadata updates, and other information. 

So at first, we tried to sync every channel, videos, and claim metadata at once every single day and it was actually possible for the first three months since the first launch of the product. But it became a bottleneck of the entire system as the # of users grew which led to lot of problems that happened at higher scale

First bottleneck, we hit YouTube's maximum daily quota and it got blocked in the mid way of sycning data. So basically the data was stale for some channels and videos, which was bad because the users were exposed to the stale data and they could take some actions against those stale data. Secondly, our Celery workers got completely saturated - we're talking thousands of tasks flooding the queue at one point which was really bads that our alerts started to trigger SIGKILL errors every morning. And our database took a massive hit from concurrent reads and writes so the whole process slowed down or even stopped taking the request. And eventually, everything became unpredictable - sometimes things worked, sometimes they didn't and we just had to look at the log to only find out it was caused by the syncing job consuming lots of memories.

So the question becomes: how do we distribute this load over time in a way that's simple and scalable.



## THE SOLUTION

So here is the solution to all of these bottlenecks. We added a single integer column to our database called "sync_bin" at video level. When a video or channel is created, a video gets assigned a random number between 0 and 1024. In terms of database schema, that's it.

Now here's the distributing mechanism. We used modulo arithmetic to create bins. The formula was: sync_bin % 7 = bin_number. So the range of the bin_number is from 0 to 6

Let me show you how this works. Say Video A has sync_bin equals 42. 42 mod 7 equals 0 - that's Monday. Video B has sync_bin equals 156. 156 mod 7 equals 2 - that's Wednesday. Video C has sync_bin 1000. 1000 mod 7 equals 6 - that's Sunday.

Because we're using random values from zero to one thousand twenty-four, we get nearly perfect distribution across all seven days. Each day gets roughly one-seventh of all videos - about 14.3% of the entire video per day.

The beauty is that it's completely deterministic. Video A will always sync on Mondays. Same video, same day, every week. This makes the system predictable and easy to reason about.



## ALTERNATIVE

We considered several alternatives. One was timestamp-based binning, basically sync videos based on when they were created. We rejected this because upload patterns are really uneven - some days have way more uploads than others basically creating hot spots.

Another was using the existing UUID column with modulo. We actually did implement this as "virtual binning" for specific use cases, basically converting UUID into integers and keep them in a separate table, but we added the dedicated sync_bin column for most syncs because it's just more performant and flexible.

We also considered a separate scheduling table consisting of video_id as a FK, columns being last_sync_at, next_sync_at, priority, etc but that felt over-engineered. The single column solution is simpler and sufficient.



## LESSONS LEARNED

2. Solve the Problem You Have, Not the Problem You Imagine
The ACTUAL problem:
* 1M videos syncing at once overwhelms the system
* and we needed to distribute the load over time or days
What were NOT the problems:
* we did not need complex scheduling with different frequencies

Sometimes, keeping it simple makes lots of things easier and maintable, without introducing a single point of failure

Lesson: we built exactly what was needed, not what might be needed someday.

Also learned that the Performance at Scale Requires Different Approaches

* 100 videos -> Sync everything at once ✅
* 10,000 videos -> Add some delays ✅
* 1,000,000 videos -> Need systematic distribution ✅ sync_bin



=====================================
Q&A PREPARATION:

Q: "What if we need to sync a video immediately?"
A: "We have override endpoints that bypass binning. They're used for emergency syncs or user-triggered updates."

Q: "What happens if a sync job fails?"
A: "Standard Celery retry logic applies. The video stays in its bin and gets picked up in the next daily cycle."

Q: "Can you change which bin a video is in?"
A: "Yes, it's just an integer column. We can manually update it if needed, though we rarely do since random assignment works well."

Q: "How do you handle timezone issues?"
A: "All datetime calculations use UTC, so the bin selection is consistent regardless of where the job runs."

Q: "What's the database overhead?"
A: "Minimal - it's an indexed integer column. The modulo operation is CPU-level and adds less than a millisecond per query."

Q: "Could this work for real-time syncing?"
A: "Not really - this pattern is designed for periodic batch jobs. For real-time, you'd want event-driven architecture with message queues and we have this in Lambda integrating PubSubHubBub."

