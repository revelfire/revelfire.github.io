<img src="../images/impossible-to-fail.png" />
# When Your Abstraction is an Impediment
On Microservices and Modernization

## This May Not Be For You!
Caveat: If you are writing SDK's, virtualization technology, large scale game engines - this content does NOT apply to you.

## Ride the Lightening
Are you doing microservices? Have you successfully migrated your complexity from your monolithic application to your highly orchestrated infrastructure mesh hosting a bunch of tiny services?

You have? Good. Now then, have you re-evaluated your architectural patterns?

Previously you had all kinds of interesting ways to handle the complexity that was engendered by your megalithic codebase. Your code was precious. You had interfaces everywhere so you could make sure you always had the freedom to insert an alternative implementation. You used JPA because hey, you just might have to move from hibernate to toplink...right? You had session facades to help aggregate all kinds of orchestrated functionality and secure blocks of your application.

Then you started writing microservices. If you really moved on, your code is now small. Your services manageable.  Bite sized even (are you doing serverless?) Heck, you reason, noodling about your flibbitywogit lambda function, if you had to you could gut the whole thing and redo it in Node or Python tomorrow, just to see if feels right.

That's how it happened right? Or are you suffering from the same feverish clenched fist death grip on the old ways that I see many colleagues entrapped by?

## People Aren't Products
The problem with tech changing so fast is that, lets face it, people really don't tend to change so fast. The "obvious" implications of new paradigms are often not so obvious. Especially to the developer on a tight deadline (always!) who has proven rinse-and-repeat patterns that they know how to build with, in go-to languages that are "production proven" and "hardened" and all that history they can leverage.

As managers or architects, how do we alleviate this road rash of high velocity tech change? We take the time to educate ourselves. To think through the real implications of such changes. To talk through these things with our co-workers who maybe aren't as eager to be breathing the rocket fumes of the bandwagon.  when we are adopting these new ideas, shifts, paradigms, what-have-you, we need to be patient. Give the devs time to learn, experiment, understand. Have them take online classes. Give them room to read articles and play with new service offerings, attend a conference. If you've got a bankroll, bring an instructor in. Start "dream talks" and let the developers take turns investigating a new tech and presenting a POC to the group.

As developers we often get excited and think things like "scala is java with funny syntax so I'll just code it in a familiar way" and poof we have a scala developer. If only it worked like this!  I've dealt with "breaking down the monolith" myself on 2 different projects now and it isn't easy to get right! Scala isn't Java, and microservices aren't just smaller webapps. We need to truly evaluate the way we are approaching what appears to be "the same old problems" keeping firmly in mind that maybe "the same old solutions" aren't a good fit - things have changed. In the current software engineering climate, what has changed is often external factors like how my service is deployed, or how I invoke other services I depend on (rest, websocket, async, reactive, rather than "included dependency direct invocation"). New patterns may be required.

## Be Part Of The Change
The specific focus of this article is: "If I am really writing a microservice, or a serverless function, am I coding in a manner reflective of that? Am I introducing unnecessary complexity out of habit?"

Here's some things to watch in your own course on any given day:

Are you using an interface where a simple implementation class will do?
Are you using an aspect to implement business logic?
Are you crafting a multi module project broken into model jars, dao jars, service jars?
Are you creating abstractions of ANY kind that will make your code difficult to understand tomorrow? 

Is the abstraction truly useful now? Or is it useful in some future world where requirements have changed and "ah ha I only have to twiddle this bit and flip the widgy and wahah!"?

In a micro and nano services world, your code should generally be more "cattle than pet", more honda accord than porche. Don't look so sad! You'll actually get a lot more done! In this world, before you go crazy and write an N-tier lasagna architecture with bells and whistles and replaceable widgets, take a moment to ask yourself:

Is my abstraction an impediment?

The answer: PROBABLY

Here's why - the abstractions have shifted around. Docker, Virtualization, Cloud Services, high level SDK's - much of what we used to need to write is now encapsulated in abstractions that we simply don't have to think about. We can focus on business cases. So - please do.






