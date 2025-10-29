## THE PROBLEM

Our platform serves over 15,000 record labels, and we're dealing with extremely sensitive data—copyrights, monetization policies, revenue charts, and more. The stakes are incredibly high here. If one label gains access to another label's data, we're not just talking about a security breach. We're talking about potential lawsuits and massive trust issues from the users. So when we were tasked with designing an authentication and authorization system, we knew we had to get this right.

The first step was defining what accessibility means on our platform. We needed to answer: who can see what, and what can they do with it?

## THE SOLUTION

We identified that we needed both **Authentication** and **Authorization**— or AuthN and AuthZ. When a user logs in, our system needs to know two things: who they are, and what they're allowed to do within our platform. 

The challenge went deeper when we considered visibility. We needed to ensure that labels could only see their own YouTube channels and videos, not anyone else's. This meant we needed an associative table, linking label IDs to channel IDs, creating that one-to-many relationship. 

### Collaborating with AuthAPI Team

To achieve this, we collaborated with our AuthAPI team—a separate team in our organization that handles all authorization-related stuff. We designed a solution around JWT tokens that stores:
- Which labels a user has access to
- What role they have (Admin, Video Coordinator, Channel Manager, etc.)
- What permissions each role has within the platform

The JWT is encoded and passed along as part of the request header. Our platform's middleware then decodes and parses the token, storing the user's accessibility information in our relational database.

Let me walk you through how the JWT is consumed from start to finish:

**Step 1: Token Issuance**
The AuthAPI issues a token using their private key, which stays secured on their side. This token is then forwarded to our platform as part of the request header.

**Step 2: Token Verification**
Our middleware intercepts the request and calls the JWKS endpoint—that's JSON Web Key Set—to retrieve the public key. We then use this public key to decode the JWT.

Now, let me talk about why we chose an **asymmetric encryption pattern** instead of symmetric encryption. In symmetric encryption, both the issuer and receivers share a common secret key. But with asymmetric encryption, only the issuer can sign using their private key, while receivers can only verify using the public key. Since we don't need to issue tokens—we only need to receive and verify them. This approach is much more secure, especially when it comes to key distribution.

**Step 3: Caching for Performance**
After verifying the signature, checking expiration, and validating the issuer, we cache the public key. Here's why this matters: the public key is short-lived. It only lasts for 5 minutes. We didn't want the middleware to become a bottleneck, so we implemented caching with a specific TTL. This way, when users log in again, we can reuse the cached key. The performance improvement is significant: fetching the public key from the JWKS endpoint takes about 300ms, but retrieving it from cache takes under 1ms. That's like a 300x performance boost.

### What This Solution Gives Us

This approach provides three critical capabilities:
* **Verify users** - Ensure they are who they claim to be
* **Label access control** - Know exactly which labels they can access and store those relationship in the DB
* **Role-based permissions** - Control what actions they can perform with their labels

## THE ALTERNATIVES WE CONSIDERED

We also evaluated an alternative approach: using Google OAuth with stored tokens: both long-lived refresh tokens and short-lived access tokens. On subsequent logins, we'd verify users using the stored token.

However, this approach had several drawbacks:

* **External dependency risk** - We'd be relying on an external authentication layer that's completely out of our control. If Google's services go down, our platform is impacted.
* **Complex token management** - We'd need additional mechanisms for hashing tokens (we do not want to store the user's token directly into our DB or cache), refreshing tokens, handling external API calls, and managing all the error handling and account issues that come with that.
* **Limited authorization control** - The biggest issue: OAuth alone only achieves authentication. It does prevent the labels from seeing other labels' data but it doesn't give us granular control over what each user can do within the platform. That's why we chose to go with JWT. It gives us both AuthN and AuthZ in one solution.

## LESSONS LEARNED

Looking back on this project, there are several key takeaways

### 1. Cross-Team Collaboration is Critical

Working with the AuthAPI team taught me the value of clear boundaries and responsibilities. Instead of building everything ourselves, we leveraged their expertise. Each team focused on what they do best, and we trusted their security practices rather than reinventing the wheel.

### 2. Understanding the Requirements Before Choosing Solutions

Google OAuth seemed simpler at first, but when we dug into our requirements, we realized we needed role-based access control, not just authentication. This actually taught me to map technical solutions back to business needs.