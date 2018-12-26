---
title: "The S-Curve"
description: "In engineering as in life, sometimes the best path forward isn't a straight line."
date: "2018-12-21T16:49:57-07:00"
categories: ["management"]
tags: ["management", "product", "project management"]
draft: false
images: ["https://enpiar.com/img/s-curve.jpeg"]
---

At [Crunch](https://crunch.io/), we've built up our product over the span of six years, and what we've built helps a lot of people analyze their data quickly and painlessly. As ever, our ambitions are great, and we want to build more. Sometimes this requires changing data contracts and APIs, and this poses an additional challenge: not only do we have to design a good solution to our users' problems, we have to get there with minimal disruption.

We're not so much ["building the plane while flying it"](https://www.csmonitor.com/The-Culture/The-Home-Forum/2016/0324/Build-the-plane-while-you-re-flying-it) as replacing one of the engines of our fully functioning airplane---and while we're doing it, we don't want to cause passenger 16C to spill her drink on the sales presentation she's preparing in flight.

Clearly, replacing a structure is far more challenging than building it from nothing, and we (re-)learned that lesson a few times last year.

{{< figure src="/img/bamboo-scaffold.jpg" class="floating-left halfwidth" attr="Building in order to build" attrlink="https://en.wikipedia.org/wiki/Scaffolding">}}

After a few turbulent rollouts, we stepped back and thought about what was missing. We realized we had been focusing so much on the "product plan"---what we were building and why it was important to build it---that we lost sight of the "[project plan](https://blog.aha.io/the-product-plan-vs-the-project-plan/)" that would get us from here to there, and that oversight became particularly problematic as we started touching heavily used APIs.

There is a lot of temporary work and scaffolding that needs to be considered in order to transition from one API to another, or to guide users to adopt a new workflow to replace old workarounds. And the product plan needs to be broken down into much smaller intermediate milestones, each with their own scaffold work, and some of which may not seem like they directly advance the product vision. The most direct path to the new feature is not the best one if you consider that current users need to be able to continue their daily work.

As a long-time Bay Area resident, a metaphor came to mind: the building of the new Bay Bridge.

# The S-Curve

{{< figure src="/img/loma-prieta.jpg" class="floating-right halfwidth" attr="Collapse" attrlink="https://www.flickr.com/photos/sanbeiji/220645446">}}

The 1989 Loma Prieta earthquake exposed many structural weaknesses across the Bay Area. Chief among them was the seismic stability of the Bay Bridge connecting San Francisco and Oakland. The eastern span was supported on pillars that did not go all the way down to bedrock, and they suffered from the liquefaction of the sea floor, famously collapsing in a few places. It would need to be replaced before the next Big One.

How do you replace a bridge that hundreds of thousands of vehicles cross daily? Very carefully. Building the new bridge next to the old one is the easy part. The hard parts are how to connect to the anchorages on each side and have the resulting new bridge have a natural traffic flow---no hard turns to drive around what used to be the old bridge.

A critical step was the construction of the "S-Curve". In the summer of 2009, workers built a structure next to the connection between the eastern span and Yerba Buena Island. The new structure would bend the flow of traffic out to the south (the S-Curve) so that the new bridge, when it was completed, could make a natural approach to the tunnel. The S-Curve also provided the points where the new bridge would connect.

{{< youtube HZ3rTq59x5U >}}

With the bridge closed over Labor Day weekend, the contractor demolished the old section of bridge and then literally slid the new structure into place. For the next four years until the new bridge opened, drivers passed over this S-Curve. Then it was demolished, along with the rest of the old bridge.

# Lessons

I think about the bridge construction a lot, and not just because I spent ten years driving across the old one, terrified that it would collapse under me. The project, and the S-Curve in particular, illustrate key points that we had been missing in our development process.

In order to replace a large, heavily-used structure, you often need to build a lot of complex, unnatural, and clever things along the way---temporary structures you'll throw away. It's not the same as designing and implementing the new structure from nothing. Building a bridge and replacing a bridge are quite different processes.

These lessons clearly mapped onto our development process, suggesting steps that we'd missed. Once we incorporated these steps, we successfully rolled out several major changes with little drama.

The following discussion is focused on changing an API, though you could generalize to any kind of data contract or promise that you want to alter. In our case, we have a web application that consumes our API, but we also have R and Python libraries that wrap it, and we have some users who directly communicate with the API outside of our libraries.

## Backwards compatibility

Backwards compatibility is the starting point. You're free to add new APIs, but you have to support both old and new for some time---a deprecation period---to allow users to update to the new APIs.

Backwards compatibility becomes increasingly complex when there are multiple layers involved. For example, if you're not only changing an API but also changing how the data is persisted, supporting the "old API" may mean writing new code that maps the new storage format to the old API. In this case, you're actually writing two new APIs: your actual new API and one that looks and behaves like the old API but is completely new code on the inside. It is easy to underestimate the effort required to pull this off.

## Forwards compatibility

While the need for backwards compatibility is generally well accepted, we found that forwards compatibility was a great opportunity to smooth the transition to a new API. If you provide client libraries for your API, as we do, you can prepare them for upcoming changes.

At a minimum, the client libraries need not to break when new API features appear, but you can do much more. We had success a couple of times this year by releasing changes to our client libraries _before_ we released the API changes. When the new APIs appeared, the library methods switched to use them internally, without requiring users to change their existing code.

At some point, you'll need to make a breaking change: whether removing the deprecated API in the end, or earlier when the new API or backend starts supporting things that can't be represented in the old API. You want this change to be as invisible as possible for your users. Even though you will need to set a hard date for this change to happen, you don't want anyone to be caught urgently needing to rewrite their code because you turned off an API they rely on, and you definitely don't want to have to coordinate with them to switch their code at a precise date and time.

Strategic use of client libraries can smooth this transition out and make deprecation of old APIs less fraught. The client library works like the S-Curve structure itself in that it supports the old workflow but is ready to connect to the new one when it is ready.

## Break work into stages

Part of Agile orthodoxy is that you should break work down into the smallest iterations that deliver value to users so that you can get feedback and refine your product vision. This is not what I'm talking about.

The stages you need for a project like this are less about delivering incremental value and more about breaking apart the complexity and disruption so that each piece is less risky. If you need to create a new API with a new persistence layer, there's a data migration involved and you need to get all API users to update to use the new API. And it needs to perform at scale. Trying to pull all of that off in one step has far too many points where it could completely fail.

Where we had success this year with major feature rollouts, the final step that released it to everyone was anticlimactic from a development perspective: all of the heavy lifting had already been done, and most of the possible failure modes had already been discovered and handled. In the bridge-building metaphor, the last step before opening to traffic is painting the lane lines, not constructing towers.

## But focus only on current stage

{{< figure src="/img/horse-blinders.jpg" class="floating-right halfwidth" attr="Nothing to see here" attrlink="https://en.wikipedia.org/wiki/Blinkers_(horse_tack)">}}

Once you've broken the grand vision into clear, self contained steps, show the team the ultimate plan at the beginning so they know where everyone is going. From then on, exclusively refer to the current stage.

If the team is looking at the product spec that describes the end goal, they may be tempted to add parts of the feature that are outside of the current stage. You very much want to avoid the "while I'm in there I'll just" scope creep. If the stages you've outlined have meaningful boundaries, there's a reason you deferred parts of the feature until later stages. Each stage depends on the previous ones being completed. If you violate those stage boundaries, all of those dependencies stack up. It turns your iterative plan into a big waterfall because now everything depends on something else before it can ship.

## Embrace the indirect route

Our development team culture values writing high-quality code to solve real problems. We don't like to "hack": we prefer to take the extra minute to think about the right way to solve a problem and make sure that we have test coverage for whatever we add. We love deleting dead code and simplifying things so that we can delete code. These are great norms, and we've worked hard to cultivate them.

{{< figure src="/img/s-curve.jpeg" class="floating-left halfwidth" attr="S-Curve in place" attrlink="https://commons.wikimedia.org/w/index.php?curid=7966067">}}

That said, as the S-Curve example shows, sometimes the process of replacing a structure means building structures that you intend to throw away. Structures that on their own make no sense---who would design a bridge that had a sharp S-shaped bend in it? It's ugly, unnecessary, and less functional than a bridge that goes point to point. For bridge-building purists, it's a horrible design.

Disposable structures like the S-Curve---or even more extreme, the support structure built on Yerba Buena Island onto which the S-Curve was constructed prior to sliding it into place---have different standards for excellence since they don't need to survive for as long. They can be hackier and uglier. They need to be strong enough to handle the usual traffic load, but they don't need to be built to last 50 years. It turned out, though, that while we like deleting code, we don't like writing code that we plan to delete. It went against our values.

It's important to know when you need a robust solution and when you need a good-enough one. One could say that that wisdom is a quality that distinguishes a truly senior engineer. Nevertheless, it was apparent that developer immaturity was not our problem because everyone, product managers and the most experienced devs included, had failed to see it. Where we lacked a clear project plan, we were on the wrong track from the beginning. We needed to expand our cultural norms to embrace the indirect route from start to finish.

# It's still software

Of course, having a thoughtful project plan that considers how to build the product safely and with minimal disruption doesn't alone guarantee success. For one, it adds work, which means it takes longer and is more expensive, and no one likes that. It does beat the alternative, but someone needs to advocate for that position and demonstrate that the costs of a good S-Curve project plan are lower than the costs and risks of powering through with the new structure as if no one was already using the old one. The Bay Bridge project suffered years of wrangling over [design](https://www.eastbaytimes.com/2004/09/23/mtc-no-u-turn-on-bay-bridge-plan/), [costs](https://www.citylab.com/equity/2015/10/from-250-million-to-65-billion-the-bay-bridge-cost-overrun/410254/), delays, and [other strange drama](https://www.nytimes.com/2012/03/23/us/a-building-ruin-on-yerba-buena-island-delays-bay-bridge-construction.html), but through all that, it was clear that shutting down the old bridge to build the new one more quickly or easily was not an acceptable option.

It's also code, which means it will have bugs, and we'll get things wrong and have to iterate on our temporary structures. That may feel wasteful, but such is software. Our temporary structures have a purpose, so they need to meet that purpose, and if they're faulty, we need to fix them---or else why bother with them at all?

As it turned out, the dramatic Labor Day weekend installation of the S-Curve wasn't the end of work on it either. There were dozens of car accidents on the curve in its initial weeks, and in November, a trucker died as he drove over the barrier on the curve. After that fatality, Caltrans lowered the speed limit on the curve, added more and bigger signs advising drivers to slow down, and installed rumble strips. [Accident rates fell](https://www.sfgate.com/bayarea/article/Caltrans-never-approved-design-of-Bay-Bridge-4773600.php) as a result.

# Be pragmatic

Finally, let's recall that not every project requires an S-Curve. It's important to understand the scope of the migration problem before designing a project plan to mitigate its disruption. Over the last decade, [California has rebuilt 647 bridges](https://www.artbabridgereport.org/state-profile/CA.html), and few if any had the combination of high daily traffic, lack of alternative routes, engineering challenges, and aesthetic concerns that the Bay Bridge had. Replacing an overpass on ([or over](https://www.dailynews.com/2013/12/18/mulholland-drive-bridge-over-405-freeway-to-open-today-after-2-years-construction/)) a busy freeway presents its difficulties and does cause slowdowns for drivers, but the options available---reducing the number of lanes available, adding temporary detours, and so on---aren't available when you're replacing a double-decker bridge high above a body of water, with a suspension span that needs to connect _just so_ to land so that traffic can pass through a tunnel.

With your code, do your homework and make sure that you don't go overboard trying to make a smooth project plan. How many users might be affected? How severely? What are your alternatives? Would bringing the service down for maintenance (assuming we're working with a web service) allow you to make your changes efficiently? If so, how much downtime would you need? Is that acceptable, or rather, how do its costs compare with a more elaborate plan that avoids downtime?

A good project plan with clearly defined intermediate milestones and plans for managing backwards and forwards compatibility throughout the process can save lots of pain on big, high-profile work. But if you can just build a new bridge and don't need to worry with S-Curve acrobatics, don't.
