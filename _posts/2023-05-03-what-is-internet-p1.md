---
title: "How does the internet actually work?"
date: 2023-05-03
categories:
  - blog
tags:
  - dns
  - learning
  - bluesky
---


# How does the internet *actually* work? Part 1

I recently got an invite to [Bluesky](bsky.app), a new social media player that aims to not only unseat Twitter as the space-king of microblogging, but also usher in a new age of decentralized social media. I've been watching the development of the [AT Protocol](atproto.com) for a while now and it's got me very interested. While it's not perfect, it definitely has a lot of strengths over the other major player in the space, [Mastodon](https://docs.joinmastodon.org/), and (crucially for this discussion) is in my opinion more easy to understand for the layman and is a great avenue for learning.

In fact, this is going to be the first in a multi-part series on *how the internet works*. The internet is something that has become so interwoven into our lives, but for many it's as mystical as the stock market or The Force. Take domains for example. Everyone knows what a URL is, but (speaking from my experience) for the most part people don’t know **how** they work and how they differ from domains. Let's look at an example:

```
 https://howley.lol/index.html
|───────|──────────|─────────|
 protocol  domain    resource
```

The above is a URL, and URLs are (mostly) composed of three parts: the protocol, the domain, and the resource. I'll discuss domains first, as those are the most relevant to Bluesky.

## Domains & DNS

The internet is comprised of servers, with servers just being computers that anyone (or sometimes just specific people) can access. Every server has a number (IP address) associated with it, which you can essentially think of as just a phone number. If you want to access a specific server (for example a search engine) you can just input that "phone number" and it'll take you right there. But numbers, especially long strings of numbers, are very hard to remember. Imagine if you had to remember that 142.250.176.206 brings you to Google, 216.24.57.1 brings you to Bluesky, 151.101.2.132 brings you to Coolmath Games, and remember another number for *every single* website you want to visit. Go ahead, try inputting the "phone number" for Google into your browser. See how annoying that was?

The early architects of the internet realized this and created **domains** to alleviate it. Continuing with our phone number analogy, a domain (e.g. `howley.lol`) is an entry in a contact book. It links a memorable name to a number. So instead of Bob having to remember Alice's phone number, he can just go look up "Alice" in his contact book and find out what the phone number is. The same is true for domains: you enter a domain into your browser, and it "looks up" what the IP is.

```
┌─────────────────────┐         ┌──────────────────────────────┐
│ Domain (howley.lol) ├─────────► IP Address (185.199.108.153) │
└─────────────────────┘         └──────────────────────────────┘
```
But what does it actually mean to "look up" and IP address from the domain? That's where DNS comes in. DNS (Domain Name System) is the contact book. More accurately, it's like a huge, kind of decentralized public phonebook. Here's a very simple example:

```
                                                          ┌──────────────────────────────┐
┌─────────────────────┐  Hey Mr. DNS, what's the number   │                              │
│                     ├───────────────────────────────────► howley.lol = 185.199.108.153 │
│                     │     for "howley.lol"?             │ google.com = 142.250.176.206 │
│ Domain (howley.lol) │                                   │ bsky.app   = 216.24.57.1     │
│                     │  Hey Ms. Browser, "howley.lol"    │            .                 │
│                     ◄───────────────────────────────────┤            .                 │
└─────────────────────┘     is 185.199.108.153            │            .                 │
                                                          └──────────────────────────────┘
      Your browser                                                  DNS Server
```
So every time you go to a website, your browser reaches out to a DNS server and asks what number is equivalent to the domain you're looking to go to. How DNS works at a deeper level is outside of the scope of this article, but if you're interested I've linked [some](https://www.cloudflare.com/learning/dns/what-is-dns/) [good](https://www.youtube.com/watch?v=uOfonONtIuk) [resources](https://www.khanacademy.org/computing/computers-and-internet/xcae6f4a7ff015e7d:the-internet/xcae6f4a7ff015e7d:web-protocols/a/domain-name-system-dns).

Yay! We can now easily reach any website through a name that's easy to remember, and that name gets resolved (turned into) an IP address. But IP addresses aren't the only records that DNS servers can hold. While IP address records (known as **A Records**) are the main thing that DNS works with, you can store a lot (and I mean [a lot](https://phoenixnap.com/kb/dns-record-types)) of different kinds of records, including aliases (**CNAME Records**), email directions (**MX Records**), and (importantly for Bluesky) **TXT Records**.

A TXT (read "text") Record can hold essentially any kind of data. They're very versatile - although [a lot of them](https://www.nslookup.io/learning/dns-record-types/txt/) tend to deal with email security. But Bluesky is using them for something else: verifying that you are who you say you are. This is known as "identity", and is done based on an electronic signature. I'm not going to get into the specifics on what an electronic signature actually *is*, but just think of it like a real, physical John Hancock that's near impossible to forge. Here's an updated version of our previous example:

```
                                                        ┌────────────┬─────────────────┬─────────────────────────┐
┌─────────────────────┐  Hey Mr. DNS, what's the TXT    │  Domains   │   A Records     │      TXT Records        │
│                     ├─────────────────────────────────►            │                 │                         │
│                     │     record for howley.lol?      │ howley.lol │ 185.199.108.153 │                         │
│ Domain (howley.lol) │                                 │ google.com │ 142.250.176.206 │                         │
│                     │  Hey Ms. Browser, "howley.lol"  │ bsky.app   │ 216.24.57.1     │ did=did:plc:pahau7...   │
│                     ◄─────────────────────────────────┤            │                 │                         │
└─────────────────────┘    is "did=did:plc:pahau7...    │            │                 │                         │
                                                        └────────────┴─────────────────┴─────────────────────────┘
      Your browser                                                          DNS Server
```

Take a look at that weird "did=did:plc:pahau7..." thing on the far right. That's the signature, the John Hancock. It's generated by Bluesky automatically when you make an account, and is linked to the data of that account - such as profile picture, posts, how many [HONKs](https://bsky.app/profile/intern.goose.art) you've done on Bluesky. And this is where Bluesky differs from traditional social media (like Twitter) and other decentralized protocols (like Mastodon). 

In the case of the former, the control over the "identity" (remember, that's like the John Hancock) is entirely in the hands of the social platform. In the case of the latter, while you do have a choice over where that identity is stored (you can choose to be on `mstdn.social` or `tech.lgbt`), you don't have complete control over it. If the specific Mastodon server you're on goes down, so too does your identity. But with the AT Protocol, identity is handled almost entirely at the DNS level, which means you can move to a different server whenever you want, while still being able to verify "hey yeah that's still actually me". And, crucially, the system will only allow **you** to post as **you**. It has a list of every signature, and if someone tries to make a post like "hey I'm howley" but the TXT Record lookup fails, the post will be rejected.

So, to summarize: URLs (like `https://google.com/index.html`) are made of three parts. The protocol (`https://`), the resource (`/index.html`), and the domain (`google.com`). In this part we looked at domains, what they actually are, how they work, and how they can be used in a multitude of ways - including identity. Next time: **protocols**. 

> Before you go, if you're interested in following me on Bluesky, you can find my account [here](https://bsky.app/profile/howley.lol). If you're interested in more technical detail (or are mad that I left out subdomains, root nameservers, etc.) I might do a series on that at a later date but right now I'm only focusing on making the internet more accessible to the everyday person. Feel free to air your grievances on Bluesky, @howley.lol.
