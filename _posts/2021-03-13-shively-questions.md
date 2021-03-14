---
layout: post
title: "Shively Questions"
---

I'm starting a new job at [Honeycomb](https://www.honeycomb.io/) soon. (No, not because of [the slides](https://ajvondrak.github.io/soapbox/2021/02/25/the-path-from-logs-to-traces/).) I've been working as a backend engineer for the better part of a decade, but this role is different: I'm going to be working as a support lead.

I can already feel the sting of that word, "support". It's like engineers are allergic to it. And I don't think I'm just projecting. When I ran the idea by one programmer friend, his initial shock betrayed his disdain. Like support is *beneath* us programmers. We spent some time unpacking it, stepping through what support would mean at a company like Honeycomb, what it'd mean for me, and so on. But I think it's telling that the gut reaction was there in the first place.

What is it that makes support a second-class citizen? If I'm honest, I can't say for sure that the disdain isn't warranted. Maybe all that "unpacking" was really just an exercise in rationalization. I don't know; I've never been in a support role before.

...Or have I? I think [Chris Shively](https://www.linkedin.com/in/cshively/), my former coworker, might argue otherwise.

<!-- more -->

## What's in a name?

As a platform engineer, I've whiled away many hours designing things, building things, fixing things. That's sort of the idea, right? In the broadest sense, software engineering is the tit-for-tat of "I program, you pay". Below that level, you have all the costs of doing business: the requirements, the estimates, the roadmaps, the reviews, etc.

But you don't just jot your code down and throw it away. It lives out there, being used by other people. Could be a lot of people or very few; external users or internal ones; humans or robots. It doesn't change your core responsibility: if you wrote it, you're supposed to know how it works.

So once you're responsible for a product or feature, you're naturally responsible for answering any questions that might come up. And that's like a job unto itself. Your mental model has to be updated continuously: what change did I just commit? what changes are other people making to the code base? are users trying to do something new? did this API that I depend on just change?

It might be a stretch to equate this tacit responsibility to support work. But then where's the line? When do we stop calling it "engineering" and start calling it "support"?

## How it starts

I was at my previous workplace for nearly 7 years, so definitely enough to see some of my own long-term trends within one environment. While I was there, that basic responsibility for knowing how things work (because I'd built them) acted like a seed crystal onto which all the other responsibilities attached. It's like the [_If You Give a Mouse a Cookie_](https://en.wikipedia.org/wiki/If_You_Give_a_Mouse_a_Cookie) effect.

For my first few contributions, what might be called the support work was barely noticeable as an extra "thing" at all. At that scale, you're just explaining how a couple of small widgets work. You're answering questions about a few lines of code, not a whole system.

Eventually, those lines of code add up. My one-off features gave way to coherent initiatives. Not overnight or anything, but in time I grew into the steward of an entire core product (the [real-time bidding](https://en.wikipedia.org/wiki/Real-time_bidding) ad server).

As you become the face of a system, the questions naturally get harder. Other engineers would come pick my brain about increasingly abstract problems. They'd want clarification not just over how *x* worked, but also what they needed to achieve *y* result, or whether them doing *z* would impact the ad server, etc.

Does it count as support if your fellow engineer taps you on the shoulder to ask about some puzzling behavior they're encountering with your API?

What about when a product person is trying to hash out a feature, but needs to know what your system is capable of handling, so you grab a conference room for an hour of Q&A?

Or say your ad server is generating low-level data that someone on your research team is tasked with deciphering (y'know, just as an example). Except they can't decipher it because they don't know how the ad server works. So you schedule several informal lunch sessions with them to go over the mechanics, which situations lead to what data, and so on. Is that support? Or is that just your tacit duty as an engineer?

## Where it goes

Okay, so I was being asked bigger questions about an entire system. But they were still all about something built primarily by me. That's just the same deal as before, right? You wrote it, you get the questions for it.

Except that it doesn't stop there. Shively made sure of that.

Chris Shively was in the business development department, which meant that his job was to create & maintain relationships with publishers. A publisher basically has a blank spot on their website where they're going to show an ad, provided that the ad server gives them one to fill in the blank. If the ad server isn't filling the blank as much as they'd like, the publisher gets unhappy.

So Shively was in just the right spot to generate the most annoying questions. A typical Shively Question™ might be of the form "why is publisher *x* seeing aggregate behavior *y*?" Finding a completely satisfactory answer was rare, and any answer usually came with much lamentation and gnashing of the teeth. What might start as a day of ad hoc investigation into the ad server might end up as a week of querying for data across a whole maze of interdependent systems&mdash;including the ones whose code I'd never even seen.

![Charlie Day conspiratorially rambling about Pepe Silvia]({{ '/public/images/pepe-silvia.jpg' | absolute_url }})

Sometimes Shively would bribe me with ice cream, though, so that helped.🍦😋

Really, any frustrations I had weren't with the questions themselves. We just liked to tease Shively. They were always really *good* questions, and they deserved answers. It's just that those answers would be hard to get. (This speaks to an utter lack of [observability](https://www.honeycomb.io/what-is-observability/), but I'm not here to shill.)

Ergo, Shively Questions could come from anyone in any department. And I'd still try to answer the ones from research, ad ops, data science, or whoever. Isn't that support work? Fielding broad questions not necessarily pertaining to your specific corner of the world? I didn't feel like a second class citizen doing it. If anything, I felt like it should've commanded more attention from the company&mdash;not making people use desserts to curry favor with busy developers.

## And beyond

Maybe answering Shively Questions doesn't count as capital-S Support work. Maybe it's more like a business intelligence role? Whatever it is, it's getting further from the engineer's core responsibility (i.e., programming). And at any rate, it still walks and talks like little-s support work, which is expected of all programmers to some extent. The lines are blurry as fuck to me.

It's okay to not want to do support work, let alone full-time. But I was paid as a full-time platform engineer and regularly did that work anyway. So it clearly isn't *beneath* programmers to do support.

You might say I was a particularly supportive engineer. I wouldn't disagree:

* By a crude `grep`'s estimation, I wrote more than 100k tokens of text in my primary API docs, which probably maps to at least 40k real words, which [qualifies as a novel](https://en.wikipedia.org/wiki/Word_count).
* I did a stint as a [university lecturer](https://ajvondrak.github.io/csupomona) early in my career, which I carry with me to this day. People would come to me as a learning resource, even for basic programming lessons on a few occasions.
* I'd regularly be called upon by QA to help explain some feature or ticket, even if I had no hand in it. They just knew I could dive in, get a read on the situation, then come back and explain it.
* And all that other shit I talked about.

I'm hoping this fuels success at my next job. I'm not sure if I'll do it long-term (I'm not about to sign up for another 7 years of anything at this point). But for now, it lets me set aside the demands of full-time programming work and broaden my experience in other ways.
