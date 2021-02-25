---
layout: post
title: "The Path from Logs to Traces"
---

A blog post linking to [the slides](https://docs.google.com/presentation/d/1GIvoN2nmXM9s2_7SJ2FWceDAG6XBvXTTTIo5FNfUNeQ/edit?usp=sharing) for a talk that (a) no one ever asked for and (b) I have no plans on giving. 🙃

But I can explain!

<!-- more -->

I'm a fan of [Honeycomb](https://www.honeycomb.io). I've been a user, [contributed major features](https://github.com/honeycombio/beeline-ruby/pulls?q=is%3Apr+is%3Aclosed+author%3Aajvondrak), and written everything from a [guest blog post](https://www.honeycomb.io/blog/using-honeycomb-to-investigate-a-redis-connection-leak) to [bug reports](https://github.com/honeycombio/beeline-ruby/issues/97) and [feature requests](https://github.com/honeycombio/beeline-ruby/issues/102).

So of course I check in on [Pollinators](https://www.honeycomb.io/blog/spread-the-love-appreciating-our-pollinators-community/) from time to time. As questions come in, you start noticing some themes. A big one is that the entire *mental model* behind tracing can be a stumbling block. But one of the things that excites me most about Honeycomb is that the mental model makes so much sense! Naturally, though, we all need someone to sit down and explain it at some point.

Feeling inspired (and perhaps a bit manic), I decided to transcribe my mental model - show everyone how much sense it makes. No one asked me to, but I get excited. It's what I enjoyed most about being a [university lecturer](http://ajvondrak.github.io/csupomona/). Channelling that energy, I also decided to write it in the form of a slideshow. It's embedded below for your convenience, or you can open it [in Google Slides](https://docs.google.com/presentation/d/1GIvoN2nmXM9s2_7SJ2FWceDAG6XBvXTTTIo5FNfUNeQ/edit?usp=sharing).

Maybe I'll transform it into prose some day. Or clean it up and actually present it somewhere. But I've already spent the past 2 days tweaking so many things that it's time to just bite the bullet and ship it.

<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vTzxkQm5bkOX22SPehbKvfI0xt7wxrPplTwCs75V89aB7cWtj_mKDvNuCugosOS93e0PoeGj_J3emWv/embed?start=false&loop=false&delayms=3000" frameborder="0" width="960" height="569" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>
