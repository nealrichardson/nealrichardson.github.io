---
title: "Writing Docs Is Better Than Having Docs"
description: "Documentation is great, but the real benefits from writing docs aren't the docs."
date: "2019-02-17T16:49:57-07:00"
categories: ["code"]
tags: ["documentation", "packages"]
draft: false
images: []
---

When interviewing developers at [Crunch](https://crunch.io/jobs), we sometimes do a live coding exercise using a JavaScript data visualization library. It's basic, and we don't assume any prior knowledge of the library. To start, we tell the candidate that they can use the full internet to help them solve the problem (we're trying to simulate real working conditions, after all), and by the way, this library has excellent documentation.

It's about 50-50 the candidates that ever open the docs---the other half go straight to Stack Overflow to look for code to copy.

{{< figure src="/img/copy-so.jpg" class="floating-right halfwidth" attr="Still waiting for this feature" attrlink="https://twitter.com/JayKuri/status/792469679486832640">}}

I make no value judgment on that, and I haven't thought too much about whether one behavior makes for a better interview. But this does seem to highlight a paradox: while everyone agrees that documentation is good and that you should write it, it's also common for people to completely ignore the documentation.

# Having docs doesn't mean people read docs

With my code, or even processes I maintain at work, when I get several questions about a topic, I figure it needs better documentation. I write it, and I still get questions. I guess it's better because I can respond by linking people to the nice documentation, which saves some time, and if I do it enough, _maybe_ they'll think to check there before asking.

But if the point of writing docs is so that users can help themselves, it clearly doesn't always work. No one likes to sit around reading technical manuals before they can get any satisfaction of seeing functioning code. I'm much more likely to pick up a new library or API, and once I know enough to get it set up and loaded, [just try things and see what happens](https://enpiar.com/2019/02/08/analyzing-logs-with-aws-athena/) and only search for help when I get stuck.

Aversion to reading the docs isn't just laziness, it's pragmatism. If you have to invest a lot of time reading docs before you can start with a "hello world" example, that's a bad sign. A library that's so complicated to get to that point is probably too difficult to use to get real work done.

# Well documented code doesn't need docs

Popular libraries and tools do tend to have good, detailed documentation. However, you don't have to spend much time reading it because the libraries are easy enough to use that you can get started and be productive quickly.

This is no coincidence. One big reason why well documented tools are usually easy to pick up is that **writing documentation makes you improve your code**. It's the act of documenting, more than the resulting documentation, that leads to a better product.

Writing documentation, like any explanatory writing, forces you to think about the thing you're explaining in a different light. Internal inconsistencies really stick out when you're writing docs. It's annoying to have to document special cases, particularly when there's no good reason why one function or endpoint works differently than another similar one. Likewise with naming things. For me, I'd rather fix the inconsistencies in the code than document a quirk that's embarrassing to explain.

This work gets you into thinking of conventions and standards in how your library or API works. The benefit of this, both to you as the developer and to your users, is that when a new function comes along, you don't have to look up the docs for it because you already know how it works. Consistency lowers the cognitive burden for everyone.

Relatedly, I've found that doc writing results in better code because **explaining how to use something leads you to make it easier to use**, particularly for new users. Several times I've been writing "getting started" guides for projects and felt that there were too many steps required, so I made it simpler. I want a user to be able to experience the value of my code as quickly as possible so that they can start exploring and being creative.

It's possible to take this reasoning to the extreme and start a project by writing the docs first. This comes in a few variations. One product-management tactic is to "[write the press release first](https://www.amazon.com/Shipping-Greatness-Practical-launching-outstanding/dp/1449336574/)." Similarly, a colleague of mine advocated "writing the how-to guide first" as a way of motivating the design. Elsewhere in software, there are those who advocate "[documentation-driven design](https://medium.com/@cmchen/-e147b1f8cd0a)". The idea is the same: if writing user-facing docs is a great way to make sure you're building the right thing, why not start there and avoid building the wrong thing first?

From a pragmatic perspective, user-facing docs are one piece of the complete product being delivered, and writing them feeds back into the product design process. Being user-oriented is essential, but writing prospective documentation is not the only way to achieve that. Docs may not be the most important or valuable part of a project to do first; often, other technical challenges and risks are more urgent to address in order to ensure the project's success. Regardless of whether one starts with the docs, it makes sense to iterate on them just as you iterate on other aspects of the product design and implementation, learn from what you write, and feed that back into the design process. So, even if they don't come first, writing the docs is rarely the last thing to do because the act of writing them almost always reveals some details that need polishing in the product.

# Not all docs are equal

When I picture "documentation", [I imagine a long index page in a dry technical reference manual](https://cran.r-project.org/doc/manuals/r-release/R-exts.html), and that's probably why "reading the docs" isn't my first instinct. But, references like this are not the only kind of documentation. [One useful categorization](https://www.divio.com/blog/documentation/) identifies four types, each with different purposes: tutorials, how-to guides, explanation, and technical reference.

Technical reference docs may be necessary for some projects, but they aren't something you should expect anyone to use frequently. They're like architectural drawings or building plans, showing where all of the walls, doors, water hookups, and so on are. You write them to make sure you have a well designed building, or at least so you know where the pipes are so that you don't accidentally cut through one in a remodel. But they're not for routine use. When you're walking inside the building searching for a bathroom, there had better be a more intuitive way to find one than consulting the as-built drawings.

The other three documentation types are more human-oriented, explaining to a person what something is and how to use it. Technical reference is necessarily more in terms of the code and its needs: what functions and methods are exposed, what arguments they accept, and what they return. With a reference manual alone, the user has to do all of the work in figuring out how to assemble the pieces.

My personal ideal for documentation would be to have an appropriate amount of short how-to guides that link to technical reference (automatic cross-linking being one of many reasons I'm a big fan of [`pkgdown`](https://pkgdown.r-lib.org/) for R), and if necessary, a more conceptual explanation-type document that serves to tie everything together. Of course, I can't say that I live up to this ideal in all of my projects. Writing helpful documentation is time consuming, and not every project gets the investment.

There's another form of documentation, often overlooked, that I find highly valuable: error messages. Good, helpful error messages are like lazy documentation, in the positive software sense of the word "lazy": they give you information only as needed. They tell you what the system doesn't allow, and hopefully they give you an indication of what you should do instead. The [Tidyverse Style Guide's recommendations for error messages](https://style.tidyverse.org/error-messages.html) are excellent. Good error messages help users learn while doing and can teach technical details without requiring them to crack open a reference manual.

# If you're so good at writing docs, could you throw them away?

Of course, you should keep them---but if you've done your job right, no one will spend much time looking at them because your product will be simpler to use. The _act_ of writing documentation and how it forces you to think about your code, package, API, or whatever, can be more important than the docs that result. The goal should be to make things intuitive enough that users don't have to wonder or ask much. When someone is told to RTFM, everyone loses.

Clearly, documentation itself does have intrinsic value. Having good, clear docs is a signal that your software is legitimate, that it is stable and popular, and that if you try to use it, it will work, as long as you follow the instructions. Docs are a kind of guarantee: I'm telling you it works if you do it this way. ([Automated tests](https://enpiar.com/talks/testing/) are the true guarantee of what works, but docs are a promise from one human to another: docs are your word.)

Having docs, particularly how-to guides and tutorials, is also a signal that you care about your users and want to help new users use your code. A reference manual doesn't have the same effect---but bothering to write a reference manual will probably make your how-to guides cleaner and simpler because in the process, you'll make your code easier to use and learn.

I don't love writing docs, but I do love making libraries and APIs easier to use. Often documentation is part of the solution but it is far from the only solution. When I start writing docs or how-to guides, I get into the mindset of improving the code so that it requires less documentation. The result is better code and lighter docs.
