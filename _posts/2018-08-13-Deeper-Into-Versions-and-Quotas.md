<img src="../images/api_versioning.jpg" />
*Advanced API Design Considerations Volume 2, with a deeper dive on Versioning, Quotas, and Throttling.*

It is likely that if you read [volume 1](https://revelfire.github.io/Advanced-API-Design-Considerations/) of this in its entirety, you were very bored that day. Or, these topics are relevant and meaningful to you in your journey to API Maturity. Hopefully the latter.

This time we are going to get into some details of how one might manage the specifics of API Versioning, Quotas/Rate Limiting, Throttling & Bursting. Sounds like elements of a good thriller film doesn't it? 

Lets start with the easy one...

## Traffic Management

We are going to discuss how to manage your traffic. The need for this might be due to:

* Making sure your servers don't tip over
* Making sure your auto-scalable services don't break the bank
* Setting limits on your client developers or other API consumers so you can sell API capacity as part of your packages
* Protection from rogue api clients/keys

If you want to implement this yourself I think that is laudable. Ways you might handle this yourself include queues, counters, latches, and a bunch of if statements in your code interrogating the inbound client api key and some counter or map of counters per key & possibly api call. If this sounds like greek, give me a shout and we can design it together. Probably I'll convince you its harder than it looks and more trouble than you want. If not, lets build it! Sounds fun.

To my thinking it is in that category of "solved problem" and would be akin to scratch-authoring an ORM. That said, ORM's are mostly free. What about API Gateway implementations?  

It is probable that using some form of an API Gateway is going to get you farther faster than building this yourself, particularly if you are doing microservices, or if you are aggregating other (non micro?) forms of services (classic service-bus stuff). Most of them come packaged in a larger ESB solution that can wind up costing millions. However, if you are careful, you can control the cost easily.

A few mature ones to look at are:

* AWS API Gateway
* WSO2
* Apigee
* Mulesoft

Mulesoft and WSO2 are available in an Open Source flavor that won't cost you until you adopt more of the ESB features or of course, purchase support or rent consultants. 

AWS API Gateway "$3.50 per million API calls received, plus the cost of data transfer out, in gigabytes" sounds pretty cheap but it depends on your use case. If you are already married to AWS it is the way to go because the integrations are already there and you don't have to purchase additional servers to host. If you are not, then it is worth considering open source alternatives. 

Of course if you have budget, Apigee is a nice full featured solution.  Contrasting these alternatives specifically in the context of what an API Gateway can provide is beyond the scope of this writing.

All of these API Gateway solutions also provide many other features that can be handy when implementing microservices. In particular, centralized authorization management, and SLA management are other related concerns that you can get out of the box with the same solution helping you with quotas & throttling - isn't that nice?

**Definitions**

Sometimes you'll see Rate Limiting and Throttling used interchangeably, other times you'll see Quotas and Rate Limiting used interchangeably.  So - to disambiguate for the purpose of this writing:

* Quotas = limiting clients by some identifier to a particular requests-per-second (rps)
* Throttling = limiting system requests to prevent server overload (or cap hardware costs)
* Rate Limiting = Quotas
* Bursting = Quotas + some forgiving threshold


Each of the ESB solutions mentioned above that provide a nice API Gateway function are managed similarly. The concepts here don't lend themselves to too much ambiguity so they tend to be approached in a common fashion. Or perhaps we all suffer from groupthink - either way, here's a few notes.

* All of these solutions allow you to manage throttling at various levels, per service, per resource, and even per method. Get as fine-grained as you need to.
* It is up to you to determine how "plans" vs. "clients" are managed, and to what level. Obviously you'll want to keep it simple to start and turn the wrench only when necessary.
* Typically you assign "api keys" to "usage plans", and set your quotas per usage plan - then apply those usage plans to a particular API or even resource. 
* AWS API Gateway feels a little less configurable here compared to Mulesoft or WSO2 as you can only apply static numbers per resource method per api stage but seriously, how often is that a legitimate use case for you to manage by-client at the resource+method level? It's still suitable for most scenarios.
* WSO2 gets really sophisticated here: https://docs.wso2.com/display/AM250/Introducing+Throttling+Use-Cases and is worth taking a look at if you have this sort of complex use case.
* Also consider if you have slow endpoints for which throttling should be more fine grained or more fixed, e.g. report generation, html->pdf, download the database to my hard drive...etc.

This brings up some interesting points to consider from the business perspective. Our our rates per api-key in aggregate, or per endpoint? Meaning if my plan says I get 100/rpm do I get 100 per resource? Per method? In aggregate? 

In general you'll want to set limits in a cascade, i.e.

* API Level (by stage/environment) (i.e. Throttling)
** Account Level (by API Key/Usage Plan/Subscription) (i.e. Quotas)
*** Resource or Method Level (Special case scenarios where system-capacity-limits are the concern) (Could be Throttling or  Quotas)

In any case, won't it be fun to finally return a `429 Too Many Requests` response rather than a `500 Internal Server Error` or `503 Service Unavailable`?


Now for the hard stuff...

## API Versioning

Why do I call API Versioning the "hard stuff"? It is not because it is difficult conceptually, the problem with API Versioning is it is difficult logistically, and full of tradeoffs.

Lets first talk about the ideal world.

In the ideal world, your API would always work with any client and all changes would be backward compatible. 

Done laughing/snickering? Good.

Change is the constant of life, and it is no different with API's. There's a few levels at which we can handle this change. 

### Routing

In the routing approach, inbound requestors are directed to discreet applications based on some inbound parameters. An important factor to keep in mind here is that with this definition for routing, we are specifically addressing an application deployment. Therefore if we say 1 version = 1 server, probably 2 versions = 2 servers and so on - so there is a direct correlation to infrastructure costs here.

Routing might be:

* URL based, e.g. https://my.cool.api/v2/user/12345
* Discreet-Header based, e.g. --header "x-api-version: 2"
* Header-Glommed, e.g. --header "Content-type: application/json?v=2"
* Domain based, e.g. https://v2-api.cool.com/user/12345
* Query parameters, e.g. https://my.cool.api/user/12345?v=2 (don't do this)

Wars have been fought over less, and to me "this is not the bridge to die on". Make your decision and make it sticky. Make sure it is well-documented.
 
I prefer header-based, because changing the URL can have impact at the code-level, as well as cause issues with security apparatus that might be based on URL paths. It can also be obscure to the developer which, if you are in a business of public API's, can brew trouble.

The most opaque to me is the 'header-glommed' mechanism, but lately its the one I'm recommending because it is leveraging an existing http header which can solve certain internet and proxy routing problems.

Of course make sure the routing tooling you have available is capable of routing based on headers and not just url pathing. Most are. But lets talk about switching, and then see if maybe a hybrid approach is in order.


### Switching

I refer to switching here as a mechanism whereby you swap code paths based on some inbound parameters. These parameters are all the exact same options as we saw in Routing, but we are using them within a single deployed application to cause different behaviors that your API evinces based on the version represented.

Whether this manifests as Aspects that marshall up a particular controller, or if-statements in a single controller, or a blend of these, in some way you are willfully spaghetiffying your code in order to accommodate an API version without increasing your deployment count.

### When to Version

You should consider your versioning requirements in the context of destructive vs. non-destructive API changes. Destructive changes are best handled by separated deployments in my opinion. If for some reason you find this objectionable, you might also consider versioning your endpoint, e.g. /user_v1 /user_v2, but that gets a bit ugly on the client side is my feeling. 

Does it matter if 5 new properties were added? Will that affect your clients? How about 2 new endpoints? Probably not. When you start changing property names, model structures/relationships, removing properties - then you need to start versioning. I would not personally choose to version an API at all unless there were material destructive changes.

### Public vs Private

Internal is easier than external. Try really hard to limit your public API change rate. 

### Recommendations

1. Use a hybrid approach, both Routing and Switching.
2. Route to major versions and host separate applications. Route via url pathing or dns. Use integers, e.g. /v1, /v2.
3. Switch to minor versions within the same app. Switch via header values. e.g. "Content-Type: application/json?v=2.1"
4. Don't version at all for additive changes, unless there is a business case for it (available to a new tier of subscription, e.g. "new insights reports"). While not strictly necessary in this business-case scenario, it has good optics.
5. Support the minimum number of historical versions, sunset often as you can.
6. Minimize the support for older versions as much as possible. Bugfixes only. No new features. Help the clients make the decision to move forward. Sometimes inflexibility is good, sometimes the customer is actually wrong (...or lazy).

### Tradeoffs

Ultimately you must ask where to best spend your capital. Do we add complexity at the developer side, via maximizing use of switching, or at the infrastructure side, by routing to multiple versions?

What are the costs of supporting Client X who "Just can't update to V2 until next year"? If the cost of hosting servers for them is greater than the value of keeping the client, fire the client, or ask for $$ for the hosting.

Maintaining multiple versions (Routing) can cost you in other ways. Process automation, CICD, QA, monitoring, support...etc. These all factor in to longer term hosting of multiple versions. Do your best to make these costs and tradeoffs clear to the business, who is most often the driver of breaking changes. Transparency at this level can save money at every level.






