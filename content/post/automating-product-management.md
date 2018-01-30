---
title: "Better Management Through Code"
description: "When becoming an engineering leader means you don't have time to write code anymore, scratch your itch by automating some of your management duties."
date: "2017-07-04T16:49:57-07:00"
categories: ["management"]
tags: ["automation", "python"]
draft: false
images: ["https://maritime.org/doc/pt/know/img/pg15.jpg"]
---


At [Crunch](http://crunch.io/), I lead both engineering and product management. Over the last few years, what that entails has changed greatly. How I spent my day when we were getting started, with a team of four developers and zero users, is very different from how I spend my time now with 16 on the team and thousands of users around the world. In fact, my day-to-day has evolved through several phases as we've grown, and each phase has come with its own challenges and learning opportunities---or rather, challenges that are learning opportunities.

# More managing, less making

{{< figure src="https://maritime.org/doc/pt/know/img/pg15.jpg" class="floating-right halfwidth" attr="'Jam sessions, razzle-dazzle cowboy stuff, and hot-shot vocalizing are all very amusing to you if you are a lunk head.'" attrlink="https://maritime.org/doc/pt/know/">}}

In this current phase, two challenges have become particularly acute. First, since the range of topics I need to coordinate has expanded, **I'm spread thin**. I work with stakeholders to set the product roadmap, with product leads to turn those needs into requirements and specifications, and with the engineering teams to get those built; and then I communicate about what we've built. (And then there's support, recruiting, and all sorts of fun things that pop up.) The more we build, the more users and other stakeholders we get, and the more developers we hire, the more work each of those becomes. That's good---I just have less time and attention to devote to each.

Delegating helps, and indeed, we recently promoted a few developers to be team leads, so I now manage managers. (That's its own [learning experience](http://larahogan.me/blog/transition-meta-management/).) Delegation isn't a cure-all, though: I still need to check in and make sure that things have been carried out as needed and on schedule. That means a lot of interrupting and context switching, which are productivity draining. Moreover, I have to *remember* that I need to follow up---that takes up too much mental space. It's easy to forget---or worry that I'm forgetting important things.

{{< figure src="https://rforcats.net/assets/img/programmer.png" class="floating-left halfwidth" attr="R for Cats---a good cause" attrlink="https://rforcats.net/">}}

Second, **I like writing code, but I don't get to do it anymore**. My time is better spent supporting the team to ship the best product, helping everyone else be more productive: in short, being a **[force](http://aboveandbeyondkm.com/2011/09/are-you-a-force-multiplier.html) [multiplier](https://medium.com/javascript-scene/the-essential-guide-to-building-balanced-development-teams-b051a62acc80)**. Taking an afternoon to write code means I'm neglecting those other needs. Moreover, insisting on contributing code would be irresponsible because I would inevitably not deliver on time---I get interrupted too much.

It's been a gradual process of becoming more distanced from the code. Early on, when I saw a server module that needed to be written and my domain expertise would help, I'd just take a day or two and do it. Those days are long gone.

Even so, both as an engineering lead and a product manager, having knowledge of the technical details of our systems is invaluable. As it got harder to carve out time to code, I might write a failing test to accompany a bug report. It would save the developer time, and in the process of isolating the bug, I could keep up with the system internals. But that rarely happens anymore.

The [Crunch R package](https://github.com/Crunch-io/rcrunch) was an exception as I was the sole maintainer. But even then, most of the additions I've made in the last year were written on airplanes, where I was free of interruption for a few hours. And as our user base grew, I wasn't flying enough to keep up with demand. Now, we've hired someone else to maintain the R package, so it too is off my plate.

I started to think about **how those two problems could be solutions for each other**. Could I write code to automate away some of my product management challenges? Could I delegate responsibilities to a computer and free up mental energy for other problems? It turns out that there are lots of opportunities to write code to make my management job easier.

# Communicating about bugfixes

One example involves letting users know when we've addressed issues they've reported. As our user base has grown, the number of bug reports and feature requests has increased. That's good! Even though they're expressing dissatisfaction, those making the reports care enough to tell us how we can better meet their needs---feedback that is essential to an agile development model. And they are optimistic enough that we will address the issues that they think it's worth telling us. That's far better than the alternative.

Just as it is important to incorporate user feedback to improve the product, it is essential to tell those users that we've done what they requested. Doing so shows that we're responsive and encourages future engagement. Plus, it's good to tell the people who care most about a new feature that it's now available---you just made their lives easier in some way.

Even though the benefits of this last step of the feedback loop are clear, we had been inconsistent about doing it. The main problem was that there's a gap between when a developer fixes a bug in the code and when that fix is operational for the user. **In that gap, we'd forget.**

Developers---at least at Crunch---don't just hotfix code on the production servers: there's a code review process, a continuous-integration system that builds and runs test suites, and then, once code has been verified and accepted, there may be a further lag in deploying the changes in production. By that time, the developer has moved on. Moreover, it usually is a support or product person, not a developer, who communicates with the user, putting extra separation between the code that satisfies a user request and the release announcement.

If we could make sure that the support team was alerted at the time that they needed to email the users, we could **solve that attention gap**. Fortunately, computers have all of the relevant information, and a means of getting our attention. All that was required was connecting the APIs of our build system, our issue tracker, and our chat system.

## The code

Lucky for me, we'd already designed a [test-and-build system](https://drive.google.com/file/d/0B5-hFeCNxubxLW9xUHBKUU1jcjg/view) that connects those APIs, so I just needed to extend that. The solution was code that, when deploying to production, scrapes the tickets that correspond to the new code, looks for markers indicating that the tickets were user-reported and by whom, and notifies our support lead in Slack that they should contact the users to let them know that the fixes are deployed.

Since our deploy job already involves a Python script, I added this block towards the end:

```python
for story in pivotal.get_stories(filter=search_terms):
    labels = [label['name'] for label in story['labels']]
    if "user-reported" in labels:
        # See if we have any users identified by email in tasks:
        tasks = [task['description']
                    for task in pivotal.get_story_tasks(story['id'])]
        users = [task for task in tasks if '@' in task and not ' ' in task]
        if users:
            users = ', '.join(set(users))
        else:
            users = 'a user'
        # Ping Chris
        msg = '@chris: %s reported "%s", which was just deployed. ' % \
            (users, story['name'])
        msg += 'Please notify them that their bug has been fixed'
        slack.message(text=msg, username="jenkins", icon_emoji=":ship:")
```

The code relies on some conventions of how we use our issue tracker (Pivotal Tracker) for user reports. In addition to tagging issues with "user-reported", we've also include the email addresses of the reporters in the "task" list on the tickets, for lack of a better place to put them. It's easy enough to do when writing the ticket based on the user's report. (We also have code that looks for email addresses of affected users based on how various other integrations create tickets, but for a more readable example, I pruned them here.)

Problem solved. Now, we don't need to remember to look up this information when a release is deployed. Moreover, the notification happens no matter what time the deploy happens, or who is on vacation, or whatever---the notice is printed in Slack for all to see. Plus, by having the job ping someone other than me, I just delegated the responsibility in the process! (Hey, it says "Please".)

# Coding responsibly

And, I got to write code! It [wasn't code that anyone was depending on](https://cate.blog/2015/12/23/the-hardest-shortest-lesson-becoming-a-manager/), and it [didn't matter if it took an extra day](https://hackernoon.com/from-engineer-to-manager-keeping-your-technical-skills-40579cc8ea00) for me to get around to finishing it. I got to tighten up our product management process and free up my mental space and that of others on the team.

So, now with even more non-coding responsibilities, I'm looking for opportunities to scratch my coding itch that are like this: things that automate parts of my job or that make things easier for others on the team. While tasks like optimizing an aggregation function in our production database are off the table, [automating our documentation site building](http://crunch.io/dev/blog/building-the-blog-on-travis/) is good. I can spend 30 minutes here and there when [productively procrastinating](http://www.chronicle.com/article/How-to-ProcrastinateStill/93959) some important task and get the high from solving a concrete problem, plus the satisfaction from making someone else's---or my own---job easier.
