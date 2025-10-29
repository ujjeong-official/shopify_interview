WHY SHOPIFY?

I believe Shopify's mission is to make commerce better for everyone which empowers entrepreneurs and small businesses to succeed globally. And to make the product at the global scale, and I believe that Engineers at Shopify are working hard to solve large scale problems. My background has been about building reliable and scalable systems and I want to contribute to a team that's building infrastructure that millions of merchants rely on daily.

And at the same time, I’d love to grow alongside with my career, tackling any challenges.



Tell us a time you had a challenge and had to ask for help. Why did you ask for help? How was the challenge resolved?

Situation:
Talk about performance issue on loading pages with 900K videos / searching videos with user permissions

Task:
I was responsible for optimizing the service to meet our latency target, but after a few rounds of profiling, I wasn’t sure whether the bottleneck was in my SQL design or the external downstream service calls.

Action:
Instead of continuing to guess, I reached out to our database engineer to review my query plan and index choices. I also asked a senior backend engineer who had worked on similar integrations to pair with me on load testing strategies.
I came prepared with metrics, code snippets, and test results so the discussion was very focused and efficient.

Result:
We discovered that the query wasn’t using an index because of one of the sub-query being incorrectly structured. After fixing that and caching a subset of the results, latency dropped by over 70%. The feature launched on time, and I learned a lot about query planning.