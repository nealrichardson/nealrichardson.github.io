---
title: "To Err Is Human; To Ignore, Divine"
description: "When you're a leader, your words carry more weight. Measure them carefully."
date: "2019-07-17T06:49:57-07:00"
categories: ["management"]
tags: ["management", "leadership", "project management", "zen"]
draft: false
images: ["https://enpiar.com/img/xkcd-386.png"]
---

One of the great things about leading a software project is the sense of ownership over what you and your team are creating. Your mind is buzzing with the big picture of what your project can do and will be able to do. You also sweat the details, and particularly if you've been with it from the beginning, you have unrivaled breadth and depth of knowledge about the project.

So, when someone---a user or new team member, for example---has a question, you know the answer. And when they make what seems like a naive proposal, you know why they're wrong. It triggers the "[someone is wrong on the internet](https://xkcd.com/386/)" sense, and you feel compelled to jump in and nip that wrongness in the bud.

Don't do it. The more influential you are, the less you should say, and the more aware you should be of how your words will be perceived.

## Your words are more powerful

It's incredibly important to recognize power dynamics. It's nearly impossible for the "boss" to participate in a discussion as just a regular team member. Your ideas crowd out others and limit or entirely shut down discussions, even without intending to do so.

I very clearly remember the time many years ago when someone on my team pulled me aside and explained this. I don't recall the issue, maybe it was the recurrence of some minor bug we thought we'd fixed, but I responded in the discussion on IRC (yes, that many years ago) with "WTF?!" My teammate quickly pulled me into a video call and said, *"You don't realize it, but you can't just say 'WTF.' It's different from when one of us says it. When we say it, we're venting frustration or showing disappointment with ourselves. When you say it, what everyone hears is that the boss is angry and that they need to drop what they're doing and respond immediately."*

Sometimes you really do want them to drop what they're doing and fix it. Usually you don't though---you're just frustrated, same as they are. But you can't express it the same way because as the leader, your words have more power behind them. You need to take a deep breath and be aware of how your words will be perceived and what action that will trigger. Think about what action (if any) you want to have happen and choose your words and tone carefully to elicit that. (Pro tip: type your frustrated reaction and delete it without sending.)

Note that you don't have to literally be the "boss" for your words to have greater power than you might think. Power relationships exist *de facto*, regardless of how flat the org chart is, where you fit within it, or how democratic and meritocratic a community may be. A product manager can have outsized influence even when the engineers don't officially report to them. A seasoned engineer can have a position of influence based solely on their experience and reputation from previous work, regardless of their official role (or lack thereof) on a project. And even in projects where all official decisions are decided by a vote, some voices are more equal than others.

At a minimum, as someone in a leadership role, you must be conscious of the fact that your intervention fundamentally changes the dynamics of a situation. A drive-by comment on a code review from a regular team member might not alter the discussion---maybe it gets a response, maybe not. But the same drive-by from the boss or project maintainer can be interpreted as a blocker, preventing resolution of the pull request until it is satisfied---even when that was not at all the intent.

When you are in a position of power, your words contain more than their literal meaning. When someone makes a proposal, and the leader immediately replies "No, I don't think so," several messages are sent in addition to the personal opinion. For one, everyone else on the team will see that the idea has been rejected by the boss, so there's no point in considering it further, even if they see merit in it. For another, the person who made the proposal will think twice about participating again---as will everyone else who witnessed the exchange. This is a vicious cycle.

These are the kinds of interactions that define the culture of a team, project, or community. Mission statements and codes of conduct are great, but culture is what gets produced and reproduced every day. People with leadership roles, whether formal or informal, have a greater effect on these cultural values, and often in ways they are not aware of and do not intend. No one sets out to lead a team or project with the goal of making a hostile environment for members or contributors, yet the history of open-source software shows how easily that can happen.

## Use them wisely

How should you act when you find yourself in a position of leadership? With great power comes great responsibility. Recognize when you have this power, use it sparingly, and make sure you're using your words for good.

Remember that your goal as a leader is to build something bigger than that which you could do alone---otherwise, why have a team at all? The bigger a project becomes, the more you must find ways to enable the team to be productive, at the expense of maintaining full control over the outcome. Once you accept that you can't control everything, the challenge becomes how to empower the team to develop ideas beyond what you could conceive of while guiding them to make decisions that are consistent with your vision.

Based on my experience, a few (deceptively simple) principles guide how you should lead. These all promote virtuous cycles that foster both the quality and quantity of team participation.

### 1. Say less

If your words are more powerful than others' words, then you don't need to use as many. Wait longer to join a discussion, and use fewer words when you do. Paradoxically, the deeper your knowledge and involvement, the less eagerly you should respond.

I feel like I've seen this sentiment expressed in a few places before, though I don't recall where---if you know of good examples, please share with me. One rather quirky version I did find is on the [R mailing list posting guide](https://www.r-project.org/posting-guide.html) (emphasis added):

> When responding to a very simple question, use the following algorithm:
>
>  1. compose your response
>  2. type `4*runif(1)` at the R prompt, and wait this many hours
>  3. check for new posts to R-help; if no similar suggestion, post your response
>
> (This is partly in jest, but if you know immediately why it is suggested, you probably should use it! **Also, it’s a nice idea to replace 4 by the number of years you have been using R or S-plus.**)

By participating less, you can be strategic where you choose to engage. Take advantage of the fact that your words are more powerful and get involved in discussions where the weight of your opinion is more important, or where your unique perspective is required.

You might fear that if you don't jump in on every discussion, no one else will, and if they do, they'll say wrong things. In fact, the causality goes the other way: others don't join the discussion *because* they know that you always do and that your opinion counts more. And they can't learn to do things the "right way" unless you get out of the way and let them be wrong occasionally.

### 2. Ask questions

When leading an engineering team, you need the team to own their intellectual work. If you do all of it for them, you're the bottleneck. Moreover, people on your team know more about things than you do, whether because you hired people smarter and more experienced than you, or because they're closest to the details of a particular problem.

In my experience, leaders are most successful when they mostly just ask questions. Questions, of course, can be leading, and you may have an idea of where you want a discussion to end up. But it's most effective when your team answers your questions and either sees the point you wanted them to see, in which case now it's their idea that they've figured out, or in the process they uncover details you didn't know about and arrive at a different conclusion, which you wouldn't have reached if you'd just told them what to do.

Either way, they have ownership of the ideas. They're not doing something because you told them to---they figured out for themselves that it was the best solution. And it helps them to feel supported by their manager or leader, that you empower them to be successful.

Another benefit of asking questions is that you're using your power to teach them what's important. You're showing them what the right questions are---that is, how to solve problems in the future.

### 3. Be positive

Think of the leader's challenge as a constrained optimization problem. The leader wants to maximize what the team produces, following the leader's vision for the project, but they have finite time and energy to enforce that vision. They can't police every discussion and support all the good ideas while quashing all the bad ones (setting aside the probability that the leader's idea of good and bad may be inaccurate), so they must choose how much effort to invest in encouraging versus obstructing.

Occasionally it is important to intervene to prevent bad outcomes, but the leader's bias should be towards promoting good ideas. Use your power to encourage action in the right direction, and ignore proposals that you think go the wrong direction. The team will learn to follow your vision, and what ideas you support and reject, based on what you choose to push forward. The absence of your support for bad ideas is generally sufficient, even without actively blocking them.

Since the leader's goal should be to foster a culture that empowers contributors, it's important to be aware of tone and how it will be perceived. Your ability to kill an idea is strong and has a chilling effect on others. Use it sparingly, and be aware of the message that is sent when you use it. In contrast, responding with positive energy to ideas you think are worthwhile promotes more contribution.

## 🎵 Let it go ❄️

Your project is like your baby: you conceived it, brought it into the world, and want it to succeed and thrive. In the beginning, you get to make all of its decisions for it, but as it grows, more and more is out of your control. As a parent, at some point you (hopefully!) realize that you can't do everything for your kid, that instead you need to model good behavior and try to teach them how to make good decisions on their own. Sometimes that means they make mistakes, but that's how some of life's most important lessons are learned.

Likewise, as a leader, it is important to let go of control. Your job isn't to do everything yourself but to help your project grow and help your team to make good choices. So, consider the lessons and messages your team will take away from your actions, and measure your words carefully.
