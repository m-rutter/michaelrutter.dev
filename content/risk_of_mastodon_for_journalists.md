+++
title = "Risks of Mastodon for Journalists"
description = "Escape from Musk but end up in a tussle with your employer"
date = "2022-11-07"
updated = "2024-09-24"
draft = false
[taxonomies]
tags = ["journalism", "politics", "twitter", "mastodon", "fediverse"]
+++

Cathy Newman of Channel 4 News, a UK news broadcaster, asks on Twitter:

> [Ok…Mastodon…erm…which server to use?][1]

The responses are a combination of mutual confusion and assurances that it does
not matter and that she can correct any mistake later. What all of the responses
fail to mention is that she might not have a choice at all. In at least one
possible future where some version of the [fediverse][2] prevails, her Mastodon
handle will be:

**`cathy.newman@channel4.com`**

This has potentially major implications for Newman's future relationships with
both with her audiences and her employers.

<!-- more -->

Nearly every journalist is invested in what happens to Twitter. It is often
their first port of call when communicating with their audiences - more so than
the newspaper, magazine, website, podcast, etc... that they call their employer.
Having a large following on Twitter is valuable not only to employers to drive
traffic but to journalists as a means of marketing themselves on the jobs
market. From the point of view of an employer, hiring a prominent journalist is
a way of acquiring a new audience. The list of Twitter followers of their new
hire figures into that hiring decision.

It is only natural that in an environment where Twitter's new ownership is
causing at least a sense of chaos and raising reasonable [questions about its
financial viability][3] that journalists are looking at alternatives such as
Mastodon.

A key difference between Twitter and Mastodon is that no single entity is
responsible for the entire thing. Mastodon is made up of multiple service
providers in much the same way that Gmail or Microsoft Outlook provides email
services. Or most pertinently, how your employer can provide email services
under their own domain. You and your followers can communicate between those
services without friction in much the same way as you can email someone at a
different email domain. It is easy for a business such as a newspaper to manage
its own Mastodon server.

On Mastodon, your followers follow you at your service provider's domain in
roughly the same way as people subscribe to say an email newsletter. Unlike an
email newsletter it is _possible_ for you to easily move to another server and
have an automated system update all of your followers to follow you on your new
service provider without your followers even noticing it happened, but **only if
the service provider is cooperative**:

> Moving your account is the same as redirecting your account, but it will also
> irreversibly force everyone to unfollow your current account and follow your
> new account, **if their software supports the Move activity**. Your toots will
> not be moved, due to technical limitations. There is also a very heavy
> cooldown period in which you cannot migrate again, so be very careful before
> using this option! - _[Mastodon - 2022-11-07][4]_

Let us imagine a scenario based on real events in the last year. [Stephen
Bush][5] is a political editor at the New Statesman. He maintains a daily
morning email newsletter that gets sent from his work email account,
`stephen.bush@newstatesman.co.uk`. Later, Bush is interviewing for a role as at
the Financial Times, a role he desires for the [larger dining expense
budget][6]. He explains that he could write a newsletter for the FT to help
drive subscriptions. While his current employer owns the list of email
newsletter subscribers at the New Statesman, he controls a Twitter account,
`@stephenkb`, with 140k followers that likely strongly overlaps with both his
current employer's newsletter and general readership. Stephen argues that he can
drive new subscriptions to both the FT and this new newsletter by soliciting his
followers on Twitter. Along with other qualities, Bush secures the job and eats
well at [St John on Marylebone][7].

Except, Bush does not have a Twitter account, as Twitter collapsed years ago.
Instead Bush has a Mastodon account that he uses for work,
`stephen.k.bush@newstatesman.com`, and which he is obligated to use for work
purposes. It does not quite have as many followers (roughly 90k) as he keeps
losing his followers with the churn of each job. He lost control of his previous
follower list at the end of his previous employment.

When Bush interviews for the FT he argues that he could drive more
subscriptions, but without being able to credibly say he can solicit all of his
followers at the New Statesman his interviewer finds this argument less
credible. Bush ends up staying at the New Statesman and cries into an overcooked
English Breakfast at a nearby greasy spoon at the next Labour Party conference.

Bush's work Mastodon account is as transferable and under his control as his
work email account is - not at all.

If something like Mastodon ends up taking the place of Twitter a best case
scenario for maintaining the status quo is that each journalist self-hosts their
own server or some independent and reliable operator(s) for journalists arises
to take Twitter's place. I think an unfortunately more likely future is that
given the opportunity by publishers to take back control over... publishing,
they will reach for control as quickly as they can to rein in their employees.

#### Update 2024-09-24:

As a relatively recent adopter of BlueSky I should say that while BlueSky's
ATProto is a fediverse type project the specific problem raised here does not
apply. The way that domain handles work there is that they leverage existing DNS
infrastructure to allow users to retain their account but "borrow" a domain from
their employer without ceding control. There might still be questions around
account portability, but that is certainly no worse than the status quo with
Twitter/X or existing social media platforms and does not involve employers.

[1]: https://twitter.com/cathynewman/status/1589384980169625600
[2]: https://en.wikipedia.org/wiki/Fediverse
[3]: https://www.nytimes.com/2022/10/30/technology/elon-musk-twitter-debt.html
[4]: https://docs.joinmastodon.org/user/moving/
[5]: https://twitter.com/stephenkb
[6]: https://www.ft.com/life-arts/lunch-with-the-ft
[7]: https://stjohnrestaurant.com/a/restaurants/marylebone+
