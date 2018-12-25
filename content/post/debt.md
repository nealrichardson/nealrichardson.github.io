---
title: "Against Technical Debt"
description: "'Technical debt' is neither technical nor debt. Discuss."
date: "2018-07-08T21:49:57-07:00"
categories: ["management"]
tags: ["technical debt", "productivity", "prioritization"]
draft: false
images: ["https://enpiar.com/img/discuss.jpg"]
---

As a product owner and development leader, I often hear that "we have to pay our technical debt" in order to proceed with our product plan. As the metaphor goes, when you choose to take shortcuts now, you incur a debt that you'll eventually have to repay with interest. The implication when a developer raises it is that those interest payments are too high and are preventing us from moving forward as effectively as we should.

{{< figure src="/img/discuss.jpg" class="floating-right halfwidth" attr="Discuss." >}}

After years of debt service and refinance, I've come to find this metaphor problematic and unpersuasive. Using the framing of debt only interferes with the real, difficult discussion about prioritization. On further reflection, technical debt isn't really debt, and the actual problem isn't technical. Even so, whenever someone blames tech debt, there is a message that should be decoded and heard.

# Technical debt is not a moral issue
<!-- argument: even when the metaphor applies, it's not helpful. calling it debt invokes morality language, which suggests priority to pay it, but origins aren't relevant and debts don't need to be paid anyway -->

Calling some work "tech debt" interferes with our ability to discuss whether it should be done.
When a developer refers to something as technical debt, they're implying that there is an imperative to pay it. This is because the language around debt in general is [loaded with moral overtones](https://www.amazon.com/Debt-First-5-000-Years/dp/1612191290): of course, debts must be paid, right?
But it's not true that something that is a debt, technical or otherwise, must necessarily be paid---and it's certainly not the case that it must be paid right now.

<!-- move elsewhere? -->
<!-- The idea that there is some ledger out there, and our project in the red right now, and if we don't balance our books, we're going to go bust (or go to hell): that sounds nice, but it's not how software works. We don't finish software by making sure all technical debts are paid. There are no technical creditors and collections agencies that will follow you from job to job, hounding you over unpaid technical debts from projects you failed to get off the ground. -->

The existence of technical debt is not inherently a problem. Let's allow that debt is an appropriate metaphor for some decisions we make in software: that means there are times and occasions when you should take on debt. Another term for debt in this context is _leverage_. You borrow so that you can do more now. Borrowing comes with a cost, of course, but that's a cost to be paid in the future, and in taking the debt, you determine that that's an appropriate tradeoff.

In economic terms, we can say that the marginal utility of future consumption is equal to that of current consumption scaled by a discount factor. Different people in different circumstances may have higher or lower discount factors, but resources you have today are more valuable than resources you might have tomorrow. Particularly when building a new product, you should assume a low discount factor: it doesn't much matter how much technical debt you have in a year if your product doesn't launch and you're out of business. So, it is good in that circumstance to take on debt and to defer paying existing debts---you need more leverage now.

Even when the technical-debt metaphor helps you to think about implementation tradeoffs, it's not helpful in the future when deciding whether it's time to "pay it off". At that point, paying the debt is just work to be scheduled---it doesn't matter that it's origins are in some past decision to take shortcuts. But when the discussion is framed with the language of debt, invoking the moral righteousness of paying ones debts, it's hard to evaluate our priorities fairly.

Referring to something a "debt" doesn't give it priority over other product demands. The language of technical debt is counterproductive for figuring out how to prioritize and schedule work, even when it accurately describes the situation.

Which it usually does not.

# It isn't really debt

By calling something technical debt, the implication is that the messy or difficult code we're dealing with now was the result of a deliberate choice, a tradeoff made for expediency's sake that may have made sense at the time, but the daily costs of that choice are now too high.

<!-- some developer in the past (probably not you) took shortcuts to get some feature out the door, and in order to maintain or extend it, we need to pay the debt and rewrite it in a more maintainable, extensible way. That is, -->

I've certainly taken on this ideal-type technical debt a few times. Of the instances I can recall where we deliberately chose to ship something early but hacky, I have zero regrets. Often, it was the Agile solution to a problem: either get something in front of users quickly to validate assumptions and guide the development of the "right way", or spend as little as possible to satisfy a peripheral need. In the former case, the prototype version was quickly replaced, as intended---short-term credit---and in the latter, any shortcomings in the implementation are well contained and don't affect us on a daily basis---long-term, low-rate bonds. On reflection, if anything I've been too reluctant to let us take on this debt.

However, none of these cases are what our developers bring up when they say we need to pay our technical debt.

Instead, when the specter of technical debt is raised, I see other origins: [premature or misguided attempts to architect](https://hackernoon.com/there-are-3-main-types-of-technical-debt-heres-how-to-manage-them-4a3328a4c50c) for some future that never came; [solutions built before the problem was well enough understood](https://einarwh.wordpress.com/2015/12/05/technical-debt-isnt-technical/); or even (especially in the JavaScript world) adherence to conventions and patterns that were standard at the time but have [fallen out of fashion](https://blog.aurynn.com/2015/12/16-contempt-culture). In a sense, [all code is technical debt](http://laughingmeme.org/2016/01/10/towards-an-understanding-of-technical-debt/).

Even the best developers aren't perfect in guessing which decisions now will turn out to be brilliant and which will be tomorrow's technical debt. While skill is involved, the bigger variable is the unknown future. A smart technical choice in one context is horribly inappropriate in another. Wiser developers may better anticipate what the future will look like for this product, or may be better at deferring expensive implementation decisions until the future becomes less uncertain, but some of their well-intentioned choices will become technical debt down the road.

Indeed, when considering these causes, much of what is called technical debt comes from _trying to avoid technical debt_. Overgeneralizing an initial implementation so that we aren't constrained by the limitations of the first version of a feature. Using the latest JavaScript framework, even though we don't yet understand it well, because it allegedly solves all the problems of previous frameworks (until the next framework comes out). And we're no more prescient when we decide that we're going to refactor or rewrite something to pay off the tech debt, so we're merely moving code around to make tomorrow's debt. Attempting to pay technical debt is often a costly refinance, in which the debt principle is unchanged, only moved to a different location.

And this is where the metaphor as it is commonly used, unhelpful though it is, really breaks down: what we generally talk about when we say "technical debt" doesn't look like debt at all. It's as if investing assets actually leads you into debt, and when you attempt to pay the debt, you just get more debt. So the metaphor doesn't help us decide how to prioritize work, and it doesn't fit with most cases where it is used.

# The problem isn't technical

So what does it really mean when developers say that they need to pay off technical debt? That they find some task harder to achieve than they think it should be due to past decisions (likely made by someone else); that the codebase (probably inherited from someone else) doesn't meet their stylistic standards; that those frustrations feel like interest payments on a debt that prevent them from being as productive as they wish; that they feel pressure to deliver more product goals, and if only they didn't have to deal with this they could meet them.

It's tempting to respond unsympathetically. We aren't getting paid to write code that's aesthetically pleasing to us: we're getting paid to deliver value to our customers. [Hell is other people's code](https://enpiar.com/2018/04/25/turbocharging-readr/#specify-column-schema), but be a professional and deal with it. But that's not a constructive response, and it risks missing the real messages. Two are particularly important.

First, they may be saying "I'm worried you're going to blame me for missing a deadline, and it's totally not my fault."
The next feature is more expensive to build than it sounds like it should be, so let's deflect responsibility for that to some external blocker, caused by decisions made in the past.

It sounds like an [excuse](http://dieswaytoofast.blogspot.com/2012/11/its-time-to-call-bullshit-on-technical.html), which it is, but the problem is that blame avoidance is very divisive and destructive.
It's a waste of energy to focus on who's at fault rather than on collaborating toward a shared goal. And it's highly toxic.

If you think this is what's going on in your team, try to understand the root cause of the (human) problem, and eradicate it as quickly as possible. For example, if your planning process doesn't include developers early enough, they may feel that requirements and deadlines are unreasonable and unfair, and appealing to technical debt is a tool to push back. So, bring them in earlier, get their input on design and costs before the requirements are complete, and use their estimates and judgment in defining project scope and deadlines. Everyone should feel like they own the schedule: designing and building a product should not be an adversarial process.

Second, they may be saying "I don't enjoy my work." When the codebase is difficult to extend, when things that seem like they should be easy are hard, or even when it just seems outdated, coding is less enjoyable. It's bad for individual morale, which then feeds into the dev culture and team identity: "I don't enjoy my work" becomes "we produce crappy work". Ultimately this frustration can lead to
[attrition](https://daedtech.com/human-cost-tech-debt/); likewise, it's harder to hire new people if you're using stale technology, and the team can't convince others that it's exciting to work here.

Thinking about it from the other direction, happy, motivated people do better work. That's reason enough to hear out their proposals for dealing with their tech debt and try to accommodate some of the work in the product roadmap. Encourage them to propose the work in a forward-looking way---how it will strengthen our product, improve some user experience, or even allow us to hire more talent in the future---rather than what past grievances it addresses. It won't actually solve the technical debt problem (because that's impossible), but if the team can agree on a course of action that makes them happier to work in their codebase and proud of their team and its craftsmanship, it may nevertheless make everyone more productive and satisfied.

# Conclusion

I may have a skewed perception of technical debt based on my experiences. Perhaps I'm fortunate to work in a low-debt codebase, and my team is full of deficit hawks who are alarmed at the thought of saddling their future selves with debt. So I won't claim that there is no such thing as technical debt. But, I think it's fair to say that much of what developers call technical debt doesn't match the ideal (metaphorical) model, and that even when it does characterize how decisions now can add costs to future work, when that future comes, the language of debt interferes with our ability to weigh the opportunity cost of "paying it off now" versus whatever else we'd do.

{{< figure src="/img/tawk-amongst-yourselves.gif" class="floating-left halfwidth" attr="Tawk amongst yourselves." attrlink="https://giphy.com/gifs/mike-myers-coffee-talk-verklempt-3oz8xsfiFFiDPjgye4" >}}

A better metaphor, if we're looking for one, might be infrastructure: it needs continual upkeep and [maintenance](http://laughingmeme.org/2016/01/10/towards-an-understanding-of-technical-debt/). Particularly as circumstances change and technology generally evolves, it is completely reasonable to keep investing in the quality of the product and its code.
If our design is proven to be unstable, it may need a retrofit.

It is essential to recall our primary goal: to build a product people want and love. It's good to take pride in our craftsmanship and to feel that quality work is valued, but how we do it and how much "technical debt" is incurred along the way is a secondary concern---one that is much more interesting to those on the development side than to the product's users. As long as the bridge stands and drivers can cross it, no one other than the welders cares how beautiful the welds are.

While the debt metaphor interferes with our work prioritization, the infrastructure metaphor may help illuminate some difficult choices we face. Infrastructure has a major "deferred maintenance" problem, in which necessary yet invisible upkeep doesn't get done. [The U.S. has over 54,000 "structurally deficient" bridges](https://www.artba.org/2018/01/29/54000-american-bridges-structurally-deficient-analysis-new-federal-data-shows/) due to this lack of maintenance. It's always more exciting (and politically valuable) to cut the ribbon at the new bridge opening than it is to reinforce old bridges.

Likewise in software, we like visible improvements to the product, and invisible maintenance work can easily be overlooked. Perhaps, though, if we think about the problem as infrastructure upkeep, we can be aware of this bias, and we can explicitly budget for maintenance. And we can evaluate design proposals on the basis of whether we expect them to increase or decrease our maintenance costs. This shift in language highlights a key truth about development that the debt framing confuses: since maintenance is a necessity of anything we build, the idea that we can pay off our technical debts---let alone that we must---is misguided at best and dangerous at worst.
