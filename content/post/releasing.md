---
title: "Obey the Timer"
description: "On the benefits of committing to a release schedule"
date: "2023-05-30T06:49:57-04:00"
categories: ["management"]
tags: ["management", "leadership"]
draft: false
images: []
---

When my kids were little, we spent lots of time at the playground. The hardest part was when it was time to leave: they were having fun and didn't want to go. When you have to drag a screaming kid off the play structure, it's unpleasant for everyone, so we needed a way to get them to come willingly.

The trick we landed on was to give them a 5-minute warning that we're going to leave, then set a timer on the phone. When the timer goes off, it's time to go. (Little kids have no concept of time, so it doesn't even need to be five minutes, for the record.) 

What's amazing about this trick is that the kids never argued with the timer. If I just told them it was time to go, they'd protest hard. But with the timer, I could be like, hey kids, I'm sorry, I'd love to stay longer, but `<shrug and point at the timer>`, timer says it's time to go.

I came to have a similar appreciation for doing time-based releases while working on Apache Arrow. Arrow does releases every three months, towards the end of January, April, July, and October. Since 2019, we've been [pretty solid](https://arrow.apache.org/release/) at that, at least with starting the release process on time. 

At first, I was skeptical: how can you ensure that you're shipping a useful product if you can't push back the release when a feature isn't ready? Releases must be just bundles of patches with no coherence, no sense of adding up to something bigger. But, I came to appreciate how the structure actually created new opportunities for ensuring that we were regularly delivering value to our users. 

First, it removes one source of energy loss on a project: debating whether we want to do a release. There's no question of "are we ready?": it's the third week of October, so we're cutting a release. The question becomes "what are we doing to be ready?" Since there's not a decision to debate, we can focus on action.

Second, it creates recurring deadlines with some meaning. Deadlines with something real on the line, like a conference presentation, are great for productivity leading up to them. Artificial deadlines like "we said in the planning meeting that we'd finish by the end of the month" never seem to have the same effect. Calendar-based releases give you a steady stream of real yet lower-stakes deadlines: real because they're public, but low stakes because the fact that we're releasing doesn't guarantee that some big feature is included, it just means it's time to release. 

If you're walking to a bus stop and you see the bus pulling up, you'll probably run to catch it, even if you know there's another one coming in a while. Similarly, if you're working on something and are aiming to include it in the next release, it's usually no big deal if you miss it, but most will push to try to finish in time. After all, the desire to build useful software and put it in the hands of others is why we got into the business in the first place. The release schedule reinforces that intrinsic motivation.

Regularly scheduled releases also allow you to get better at releasing. Writing beautiful code isn't enough if you want people to use and depend on your work. Things like packaging, verification on different platforms, writing good docs, and publicizing through blog posts and other annoucements are important yet easily overlooked. For many, these tasks seem burdensome, so we avoid doing them, but that means we never get better at them. Regular releases force us to develop those muscles. 

Finally, as a manager, I found the regular release cadence to be useful for providing guidance to the team about what to focus on when. Explore new ideas early in the release cycle when there's more time until the next release. As the weeks go by, the scope of project you want to pick up should get smaller. The final days or weeks are all about finishing things rather than starting new ones. Right after a release is a good time to knock out all of the little issues you noticed when pushing to get the release out but didn't have time to address before. And then the cycle repeats. 

This has a nice unifying effect in an open-source project where everyone may be working on their own features or backlogs. We all share the release calendar, and we need to support each other to make the release a success. 

Scheduled releases can be helpful for both projects in active development and those in maintenance mode, though of course with different frequencies. For active projects, a steady cadence of releasing and announcing creates a drumbeat of progress, a sense that the project is continually improving. For less active projects, it's less about showing progress as about not forgetting to release bugfixes that have accumulated. 

Either way, save yourself some trouble by setting a regular release schedule---even little kids know there's no use arguing with the timer.