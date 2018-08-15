<img src="../images/api_design.png" />
*Volume 1 - The Broad Strokes*

So often we are at the ground floor of API design, discussing (arguing?) basic principles, usually around REST and "is it RESTful, is it right?". Some of the API world has moved beyond this, building a complete abstraction of RESTful principles and creating a whole new way of addressing remote entities - GraphQL being the notable loud-voice of late. 

In another case, HAL, HATEOAS, JSON:API - all approaches for considering the REST API in the same way we might want a webpage - as "hypermedia" that amounts, basically, to "list of links to lists of links". The basic notion is "don't just give me a payload, give me a payload that tells me how to get to the next payload I might care about" - such as pagination links for a listing, or links to detail records of child-objects in a parent-object I'm looking at (e.g. customer::address).

In their own right, these approaches qualify as "advanced API considerations" - but I'm going a different direction with this.

I've been in both camps a bit, and played around. There are some serious conveniences for the API consumer, to be sure, particularly in a single-page-app front end context. On the other hand, implementing JSON:API or GraphQL require some pretty heavy duty backend coding and thoughtful architecture. It's not your "expose a resource on the web" baseline. I'm not arguing for or against them; only saying the complexity introduces investment. On the completely opposite side of the spectrum is something like Spring Data Rest, where they make it stupid simple to throw a Dao right on the web as a REST API. You'll make a lot of concessions, and probably jump through some hoops to get everything you need, but the basic setup is like ruby-on-rails-make-your-own-blog-demo simple. (If y'all recall that era.)

As someone who came up in the days of the SOAP protocol, I find all these attempts at formalism an ironic conclusion to the "wow REST is so much easier" saga.

So lets say, for arguments sake, that like many shops you find these big ideas interesting, but would prefer not to let them get in the way of actually getting stuff done. Lets say you just want your resources exposed so you can get on with your actual work. Here are a few things you'll want to keep in mind as your baby grows to a toddler, and hopefully, into young adult-hood. This is by no means an exhaustive list, just some things I've been working with, researching, and thinking on of late. This is all in the vein of "designing for success".

## Intentional Limitations
There are a few categories to consider here. Lets look at the use cases.

Your system has a well-known maximum and you need to protect it.  You will use the enticingly termed "throttling" - better stated as "Rate Limiting". This involves some measure your system, typically Transactions Per Second (your TPS Report, sir). Whether done per-application or on a per-client basis, the implementation provides a mechanism to monitor the throughput and physically slow down the processing by reducing/limiting the number of concurrent activities. This likely will involve a queue as part of that mechanism to both know what is in flight, and what is pending, and provide some traffic management.

Your system is now rate-limited but certain special clients need extra boost. You will use API Burst, a mechanism for allowing additional capacity for a particular client, over a short period. This bursting can be handled by allocating a timeframe, additional hardware, or running your regular rate-limiting at a level where your system is tolerant to the burst cases.

You want to meter your system, and provide consumption-cost structures that clients pay for. You will implement an API Quota, wherein each clients usage is recorded and compared to their allotment. When they begin to go over this plan-imposed limit, you can send a fun { err: 429, message: "Aww! You ran out of quarters. Please insert more." }

All of this can take quite a bit of work and is easy to goof up, consider using an off-the-shelf API Gateway.

## Contract Ownership, Swagger & Static Dev

Ok this one isn't exactly an "advanced design consideration" - more of a "best practice". Lets be realistic though - most of us just get ordered to start coding and get it done as fast as possible. Thus do we break the "good" leg of the "good/fast/cheap" triangle. 

That said, lets do something about it! There's so many great tools out there these days. Here I will make a few statements, and hope not to start a war.

* For internal API's, the front-end developer owns the contract. This doesn't mean the backend guy doesn't get a say - they are involved, but not committed. 
* Swagger is mandatory for "regular old" RESTful API's. It says all the stuff that needs to be said, reasonably tersely (I prefer YAML). It is majorly interoperable.
* Once we have a Swagger file, we generate a client. Use swagger.io, amazon api gateway, or any of a hundred other tools. I like swagger.io. I can generate a server and a client in at least 10 different languages.
* This client sometimes can stand in as a starter for the real client. Sometimes not. At least look at it.
* This server should remain inviolate, and be generated as-needed for a given API version. This server can be used to build out the front end, demo to stakeholders, and very quickly iterate. 
* You can use apiary or many other tools if you like someone else to host the code.
* You can use a tool such as RoboHydra to switch-proxy contents between your mock API and your real API in cases where you need a split-brain for part-real and part-mock setups.

In summary - choose a contract owner, your best results will generally be with the actual consumer owning it. More than one? Give them all a say (if they are internal) - often, mobile doesn't have the same needs as browser. Create swagger. Use static API/generated server. 

Then let the backend guys start coding it up. This "building the walls first" approach can really avoid a lot of pain.

## Versioning

API versioning. Everybody hates it. Am I right? Anybody?

Seriously though - it's as necessary as it is painful. There's a few common ways this is done, and some pretty simple best-practices. The hard part is usually implementation.

### Access
We need a way to tell the server which API we care about. This is often done through url-versioning - e.g. http://api.foo.com/v1/endpoints (best practice note - stick with integers, and preface with a v).  In other cases this is done with http request headers, eg. x-api-version=v1. My least favorite approach is in the content-type header, e.g. Content-type: application/json?v=3 or some variant thereof. That one just annoys me. It's also the one I tend to use. More on that in Volume 2.

Each of these has its peculiarities, and the arguments for and against them run the flavor of ideology rather than pragmatism. Pick one and stick with it. They'll all work.

### Routing
You'll want to figure out how to get to the right version of code based on whichever versioning scheme you pick. That's where it gets tricky. Do we do this at the routing layer? That might imply that we have different servers/services hosting each version. This means my codebase is probably cleaner but I'm likely paying more money for infrastructure.

So maybe I'm routing in the applications. This means my code is getting muddy and possibly tasting a bit like spaghetti - so I'm spending more on dev instead of on infrastructure Argh!

There's quite a bit more to this. The complexities of models changing almost requires putting up different versions of code - an added property here and there is fine - but if you end up restructuring how you'd like your consumers to see the data, you will end up with terribly complex and unmaintainable code. More on that in Volume 2.

### How Many Versions?
I suppose this is a business question. I like the answer "current - 1" because that makes the preceding paragraph a bit easier. I can have a pretty finite infrastructure cost if I can guarantee that I only ever host 2 versions. Inevitably though business will find reasons to host more like "That 1 freakin guy we don't want to piss off that refuses to update his app" or "the partner company that uses the API that is 3 versions old and they lost their developer and can't update till next year". There will always be a reason. Like anything, it comes down to finding a good balance.  Internal is easier than external. Try really hard to limit your public API change rate.

*What if No Version is Specified?*
Show the latest.

*What if That Screws up Customer X?*
Meh. Teach your users to always pass a version.

## Coarse vs. Fine Grained

Turns out this is a pretty hotly contested topic. There's a (large) camp of backend developers that feel as if their whole job should be to expose the database on the web. Let the front end devs or api consumers figure out the rest. This is often the argument for fine-grained API's. It's the "teach them to fish" approach. The trouble with this approach is that while yes, access to all those fine-grained endpoints for every model entity in your API is good, clean, simple to understand and build, it can really backfire. How many business rules can be circumvented unintentionally here? Just give them the fish, for goodness sake.

My argument is yes: do fine-grained API's, but only for internal consumption, and secure them as such. Maybe they are a second layer backbone in your microservices. Build Coarse-Grained API's too. I've also used the term "orchestrated" API's. In the 'old days' we might have used the term Session Facade Pattern. The basic idea is to take the primary aggregate cases in your application, and sequence them in an API that takes the necessary steps to include the various services that know all the business rules.

Here's an example. Do you want the front end developer or public api consumer to 

1. Create an Account
2. Create a User, manually associating the account id
3. Create a Profile, manually associating the user id with the profile
4. Create Entitlements for the User
5. Log the User in

Or do you want an API called "provisionNewUser" that takes a few pieces of info and properly orchestrates that series of calls?

If you are on the consumption side, I can predict which approach you might feel more affinity toward. Model your API's after your primary use cases and reduce the heavy lifting. Expose your fine-grained API's surgically where necessary to accommodate corner cases or internal needs.

## Data Sourcing & Freshness

Do you need an Asynchronous API? What is an Asynchronous API anyway? There's async implementations, like Node or Vertx. If you expose an API with Node, is it async? Well - kind of - from an implementation standpoint. But no - typically you still have request/response cycles on a synchronous wait-cycle anyway. This is not "async" from the API/consumer perspective. 

However, if all your inbound messages are "fire and forget" and you "respond" by eventually invoking something else - maybe pubnub, or a websocket stomp response message - or maybe your client doesn't even require a response at all (think event collection), then yes - you are building an async API.

I don't buy the "reactive" assertion that "everything is a stream". Reactive (RxJava, ReactJS...etc) sounds good on paper. Then you build something. Then you debug it, painfully. Then you run it in production. Then you troubleshoot it, more painfully. Async request/response is a pain, no two ways about it. The best kind of Asynchronous API is the one whose clients don't care about the response; if you are receiving streams of data, for example. The client only cares that it landed, not what you did with it. 

Often the main reason for this whole Async/Streaming approach is to sell the notion of "real-time" - real time analytics, insights, real time snake oil - whatever it is. While the term "real time" is a bit silly, the general idea here is "as fast as possible", do some processing that matters to a core business case. It's also nice when the business case aligns with the requirement to happen faster. If it's your daily revenue reports, then that real time infrastructure didn't much matter. If it's your responsive ad bidder reacting to impressions and conversion rates dynamically shifting your ad spend, then yes, "real time" is real darn handy.

One phrase I've heard at times, that I enjoy the sentiment of: "Data is always old. It's just a question of how old."

Another topic that often gets conflated with Asynchronous in part because there really is some overlap, is Push vs Pull. This relates to mechanisms that consumers can use for interacting with you. The obvious case here is CRUD. You'll usually want a "push" API for the crud use case - that situation where your consumers are using you as a RESTful datastore. Sometimes though, in API->API communication, you'll find it useful to manage your own data store by pulling from another with some processing logic. Maybe you are syncing up with a SAML provider and using content from that central store to update your internal knowledge of users. This is still a form of CRUD use case, but one that makes sense in Pull context. There are really no hard fast rules. Better to understand common mechanisms, and make use-case-based decisions.

If we're still talking about data freshness, then we get into some of the overlap with Asynchronous API's. Some options here for trying to stay current with data include:

* Polling - on a cyclic basis invoke an endpoint to see if anything interesting happened. This is kind of the exact opposite of the stream-based approach. It's inefficient, but simple, and sometimes, just enough.

* Push/Webhook - In this case our consumer registers an endpoint with us that they host. We post something to this endpoint you when something they said they cared about happens. Alternatively, they register a common endpoint for all updates and parse/decision in their handler how to disposition the various updates that come through. This works really well in many situations. Credit card processing with Stripe uses this mechanism to let you know all kinds of interesting events - when a user purchases something, when a card expires, when a subscription is due. All things happening on different timelines - imagine the polling schedule to accommodate the cases here! Arguably, this Push/Webhook approach has been a key to the success of SaaS solutions in the API economy.

* Dynamic Subscriptions - Here we let our consumers tell us - resource by resource - which resources they are interested in knowing about updates to. This can be an optimal solution and is quite similar to the Push/Webhook model, but with the added benefit of reducing chatter by allowing more fine-grained control of what gets sent to the listener. While less common and slightly more complex, this can be a great solution also.

## And Now
If you made it this far, you are amazing. Thank you! I feel these considerations are often push to the backburner. Maybe that's ok. Maybe they don't matter until they do, and usually that can be correlated to success of the API/business. If your API is becoming an adolescent, then hopefully I've said something useful here.  This is obviously a very high-level overview, and perhaps I'll do a few deep-dive pieces. I think soon there may be a Volume 2, with a deeper dive on Versioning, Quotas, and Throttling.



