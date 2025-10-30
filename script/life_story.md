So I guess I can start by sharing how I got into the tech field.

I think my journey into tech actually started pretty early, 
umm I was one of those kids who’d spend hours gaming alone in my room, 
but instead of just playing, I was always curious about how things were working behind the scenes, 
like how data moves, how systems interact, and if you look at one of those famous online RPG games like World of Warcraft, how games manage so much happening at once without crashing 
and that curiosity is really what pulled me into tech.



HIGH SHCHOOL (OPTIONAL)

So ummmm  Back in high school, there was a computer science course that the school offered for an entry level programming and I wanted to see what programming actually looked like, so I enrolled without knowing anything (my gut was telling me to do it, it wasn't me) — Now I think about it, it was my first real exposure to computer science. The language we used was Java, and I still remember writing my first “Hello World” program.

The challenge, though, was that almost everyone else in the class already had some programming experience, while I had zero. The final project was to create a game using Java, and at the time, I didn’t even know what a loop or variable was. It felt overwhelming at first, but I was kinda determined to figure it out.

I started learning on my own — spending hours on online resources to understand the basics and searching Stack Overflow for specific issues I ran into. I often stayed after class to get feedback and guidance from my highschool teacher.

In the end, I built a shooting game with multiple levels, where the difficulty increased as you progressed. It was incredibly challenging, but also one of the most rewarding experiences I've had



UNIVERSITY

So when it came time to choose a major, I went into System Design Engineering at the University of Waterloo. 
I picked it because it blended the analytical side of engineering with problem-solving and design thinking. 
But the program didn't go super deep into computer science 
so I realized if I wanted to understand how real-world systems work, I'd have to learn a lot on my own. That's when I started experimenting with code outside of class, and I set my sights on landing software engineer roles for my co-op terms. 

By the time I graduated, I'd done six co-ops — five of them as a software engineer.
So that hands-on exposure to the software field was what made everything click for me in my early career. It sort of taught me that I like building systems that don’t just work, but the system needs to be scalable and reliable.



FULL TIME CAREER

MODERN CAMPUS

So after I graduated, I started to search for a software engineer roles in Canada to continue to see what I can achieve in the field.

My first full-time job was at the company called Modern Campus, it was a company building CRM platforms for universities and colleges. They were building a platform helping the clients manage their students, course enrollment, proctored exams, and certifications. 

I started in a Support Engineering role, fixing production issues and designing sustainable fixes. 
It might sound basic, but it taught me a lot about empathy for the user and understanding how software behaves in production.

After about a year, I moved into the Core Feature team, where I got to work on building features end-to-end and think more deeply about scalability including database design, APIs, queues, and all the moving parts that make a system reliable. When I joined the team, it was specifically the payment processor that was quite fragmented. Uhm Institutions had limited options and frequent payment failures due to inconsistent third-party integrations. So I took the ownership of that area and enhanced the system to make it both scalable and reliable.

MODERN CAMPUS IMPACT

To ensure the consistency at scale for the payment processor, I implemented idempotent transaction handling on every charge/refund so any duplicate requests return the original result instead of double-charging which could crucially impact the colleges financial reports. I also implemented asynchronous retries with exponential backoff (You don’t retry immediately in a tight loop, instead, we space out retries with increasing delays), and some webhook verification, for example we used HMAC (symmetric encryption - both sides use shared key) - it’s a cryptographic method that proves a message came from a trusted sender and wasn’t changed in transit. Basically included x-signature in the request header. We used this mechanism to gracefully recover from partial OR duplicate payment states. After all these implementations, I integrated multiple major payment gateways including PayPal, Bolt, CardConnect, Payeezy, and Flywire through RESTful APIs and these payment processors were efficiently working with all thoes features we implemented.

We also added structured logging and audit trails across all payment flows, which made the reconciliation and debugging far easier. Back at the time, the result was a around 35% increase in monthly online payment usage, along with a huge drop in failed transactions and support tickets. One of the key insights here is that there wa a huge drop in the # of failing transactions, which led to building a user's trust on our platform.

That project really taught me how to think about distributed reliability and financial correctness — not just getting a feature to work, but ensuring it works safely at scale.


MODERN CAMPUS CHALLENGE

One of the challenging part while working at Modern Campus was that the core platform was built as a large Java monolith with the server on prem, with a JSP + jQuery as a frontend techstack and SOAP-based web-services.
Even at that time, these technologies were considered legacy, which made new feature development very slow and integration with newer APIs difficult. The system was stable but not flexible / extensible, and we often faced challenges around code maintainability, testing, and onboarding new engineers into this gigantic monolith system.

At the time, our goal was to improve developer productivity and system reliability without breaking the legacy foundation that hundreds of college clients relied on.
Specifically, I needed to introduce more modern patterns and tooling to make development safer, faster, and, you know, easier to extend while minimizing risk to production systems - we were in a very tricky situation.

So basically to address this situation, I modularized several parts of the monolith into independent services, building RESTful APIs from scratch all the way to deployment alongside the existing SOAP endpoints so clients could transit gradually.

These changes ended up making a really big impact. As a result, our overall development velocity and reliability improved noticeably. The modular boundaries made it much easier to add new business features without worrying about breaking existing ones, which led to regression issues dropping significantly, I personally believe this wa a big win since there were reduced # of support tickets coming in as a regression issue.

Overall, what used to feel like a fragile, the legacy system started to evolve into a much more sustainable and maintainable platform.

I spent nearly five years there with lots of talented engineers in a collaborative culture. It was an amazing place to build strong fundamentals and definitely it was an amazing ride in my early fulltime career in the software field.


WAYFAIR

Around the five-year mark, I started feeling like I was ready for the next challenge - 
I guess I really wanted to see how engineering looks at a much larger scale. 
That’s when I joined Wayfair. 

At Wayfair, I worked on the Bidding & Optimization Platform that powers automated bidding for billions of products across vendors. So the system we owned, it processed roughly 1.3 billion SKUs every day, generating optimized advertising bids using trained models. 
And My day-to-day involved building and optimizing ETL pipelines — 
for example, like taking raw click data from external ad aggregators, transforming it, and using it for model training and real-time bidding. So the scale there was totally different.

WAYFIR IMPACT 

My work had a direct impact on both system performance and operational efficiency. Firstly, I led the modernization of our ETL pipelines. I migrated them from Hive-based batch queries, what we used to have it as a legacy, to a Spark and BigQuery–powered architecture. It sort of enabled distributed in-memory processing and much faster query execution. The result cut end-to-end data processing time by roughly 80% and this was a huge impact — from 4 hours to just 30 minutes.

Within the Bidding and Optimization platform, we also had a compontent called a Bid Uploader and the purpose of this application is to upload the optimized bids to the vendors including Google, Bing, and Facebook Ads at the end of the streaming process so that the genereated bids are actually applied to the online product. The Bid Uploader was also an out-dated technology at the time - it was really slow in collecting and uplaoding the bids which consumed around 3.5 hours to upload everything to the vendors. 

So the legacy Bid Uploader was a long-running monolith that pushed all vendor bids sequentially and even it often failed (batch failure) mid-run.
So I took the entire ownehership in that area and redesigned it using uhhhmmm Airflow as our DAG, FastAPI, Kubernetes, and CloudSQL.
Airflow orchestrated each stage in parallel, while FastAPI handled async uploads per vendor.
We stored state in CloudSQL for safe retries and deployed everything on Kubernetes with auto-scaling. We used CloudSQL as our primary storage for Bid Uploader, where one of the purpose was to store the job-state while processing the upload to each vendor. So each vendor upload had a record with its stage and status, and FastAPI would update it transactionally after every step. If a job failed mid-way, the pipeline could resume exactly from the failed vendor instead of restarting the entire run.
As a result, that reduced the total runtime from 3.5 hours to just 20 minutes for over 400,000 bids a day, and made the whole process traceable, fault-tolerant, and scalable.


WAYFAIR CHALLANGE

I guess one of the challenges At Wayfair, there was a company-wide cost-cutting initiative, and every engineering team was asked to come up with a concrete plan to reduce cloud infrastructure expenses without sacrificing performance or reliability. It was a tough ask.
For our team — which managed the Bidding & Optimization Platform processing billions of SKUs daily — this was particularly challenging because our workloads were both large-scale and performance-sensitive.
We were actually identified as the second-highest cost-consuming team across the organization. Given the nature of our system — which relies heavily on memory-intensive data processing and model training — the spending was somewhat justified, but there was still clear room to optimize and be more efficient.

So I started to dig into our stats. My responsibility was to identify where we were overspending in compute and storage, and to design a replatforming strategy that could lower the costs while maintaining the same SLA.

I started by doing a detailed analysis of memory and CPU utilization across our workloads (just an fyi we used Datadog for metrics), which actually revealed a lot of inefficiencies. We were over-provisioning resources for certain ETL pipelines and model-training jobs.
I started to migrate our legacy monolithic services onto Kubernetes so that it became a micro-service structure where we could leverage auto-scaling and right-sizing policies based on the actual utilization metrics. And we started to use the Cloud Run which is a serverless computing and it's pay as you go.
Along the way, I worked closely with the DevOps to tune the container limits, optimize scheduling, and automate resource monitoring so we could continuously adjust capacity as demand changed.

Through that effort, we were able to reduce our annual infrastructure costs by roughly $400–600K, while keeping the same performance for the same data throughput.
I guess it was a great example of driving both technical and business impact. We were able to improve cost efficiency at scale without breaking the quality or speed of our systems, in fact we actually improved performance by changing the monolithic structure to micro-service based.


WAYFAIR LESSON LEAERNED 

One big lesson I took from that project was the importance of the data-driven optimization.
Instead of guessing where to cut the costs, I learned how important it is to use the real-time utilization metrics — like CPU and memory — to drive engineering decisions.

Another takeaway was how much cross-team collaboration matters. Working closely with DevOps, I realized that cost efficiency isn’t just about better code. It’s about aligning engineering, infrastructure, and product priorities.

And lastly, I learned that engineering constraints can coexist with business goals. You can deliver real cost savings without degrading performance, as long as you measure, validate, and iterate carefully.

It was fun and every tasks were rewarding at Wayfair. The only tough part was the volatility during 2022–2024 — 
I believe at that time, the global economy was rough, and Wayfair went through multiple rounds of layoffs. I was just fortunate enough to stay through them, but seeing teammates go really changed how I thought about long-term growth and impact at Wayfair. It made me reflect on what kind of environment I wanted to be in — 
I really wanted the one where I could build something more stable, with a longer-term product vision.


WMG

That led me to Warner Music Group, where I am now. WMG is a huge record lagel that manages thousands of contracted-labels under them and provide management tools so that we can help the sub-labels making the profit out of their content
I lead a small team within the project called Video Claiming 
where we build an automation platform that helps record labels manage their YouTube channels, videos, and content-ID claims. Well basically, the ultimate goal is to help labels monetize their content and protect their rights which also contirubtes to the company's revenue growth.

WMG IMPACT

Within the Video Claiming project, I designed and implemented large-scale data-sync pipelines that now handle over 8,000 YouTube channels, nearly a million videos, and more than a million claims and assets.
These pipelines consume around half a million YouTube API requests daily, keeping metadata and ownership data fresh across our internal platforms.
One of my key achievements was cutting YouTube data sync times from 6+ hours to about 30 minutes and improving the query latency by 50–70%. I can further extend on this context as I go.

This platform is heavily dependent on YouTube API. We would need to periodically sync millions of YouTube data into our Database so that our users always have the access to the latest data.

I did this by batching API calls to reduce request overhead, running parallel ECS workers to process multiple channels sync concurrently, and optimizing SQL queries with denormalization, indexing techniques, converting data types into more efficiecnt data type in query (i.e. integers instead of UUID strings), and applying B-Tree (for prefix matching) / Trigram (for fuzzy words) GIN indexes for faster lookups.
These changes shortened the sync delay from several hours to about 30 minutes, so our internal systems now reflect YouTube’s data almost in real time.
As a result, the record labels could detect the ownership of the claims or monetization changes within the same day, and analytics teams could make decisions using current, not stale, information.

What I found interesting about this project is that it started as something small — a short-term MVP — 
but as I got involved in stakeholder meetings in the early phase and scoped out what was needed, it evolved into a full product initiative.

WMG CHALLENGE

There were tough moments too — during the early phase, the stakeholders wanted it fast, but the reality was we had engineering debt and limited resources. 
I put together a project roadmap with clear data points — 
what was blocking us, what resources we had, and realistic milestones. 
Once everyone was aligned, we got buy-in from the leadership 
and we were able to scale the project into a long-term which now became a fully supported platform.


WMG LESSON LEARNED

The biggest lesson I learned from this project was the importance of transparency and alignment.
Early on, everyone wanted quick results, but what really made the difference was stepping back, laying out the facts, the blockers, the resources, and what was realistically achievable, and getting everyone on the same page.

I realized that when you clearly communicate why something takes the time and show a data-backed plan, people are far more willing to support you.

As a result, the platform is well used by thousands of labels worldwide. 
Seeing that growth — from an early MVP to a global-scale tool — 
has definitely been one of the most fulfilling experiences at WMG.

And now I’m looking for what’s next. 
I’m ready for another scale jump — 
something where I can apply what I’ve learned about building reliable, scalable systems 
and leading projects that have real user impact. That’s exactly what draws me to Shopify.

I believe Shopify's product is operating at a global scale
and I want to bring my experience designing systems for reliability and scale, and apply it to a platform that literally powers millions of merchants around the world.

So that’s kind of been the thread through my whole career — I guess, curiosity leading to challenge, and challenge leading to growth eventually. 
I’ve gone from wondering how systems work behind the scenes, to building and scaling them myself — 
and now I want to keep doing that, just at the next level of global impact.






Q -
 
How is the team structured, and how does it collaborate with other teams?

What's the balance between building new features and maintaining existing systems?

How much autonomy do engineers have in choosing technologies or architectural approaches?

Given my experience with cost optimization at scale, are there opportunities to work on infrastructure efficiency here?










