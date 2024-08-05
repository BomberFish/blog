---
tags: Devlog
---
# Proxying on the Edge: Part 2

## Recap

Recently, I made a proof-of-concept edge proxy called Superposition. It was very unpolished, and served only as a proof-of-concept.

## Infrared

In [my last post](https://blog.bomberfish.ca/2023/09/25/cf-workers-proxy-part-1.html) on this topic, I mentioned I was working on a new proxy called Infrared. I am proud to announce that I have officially released it, and you can visit it at [infrared.bomberfish.ca](https://infrared.bomberfish.ca). This post will explain the challenges behind creating a proxy like this.

## TompHTTP

Since my last proxy was so unpolished, I made sure Infrared conformed to a popular proxy standard. Thankfully, the [TompHTTP](https://github.com/TompHTTP) standard proved popular enough. 

## bare-server-worker

It turned out that work was already done for porting the TompHTTP bare-server to the Workers runtime, called `bare-server-worker`. However, this implementation was not maintained, so it only supported the older V2 standard.

## Adding V3

Adding support for the newer V3 standard was trivial, as the only major change was merging the old URL segment headers into a new header, named `x-bare-url`.

## Frontend

I opted to use [Ultraviolet](https://github.com/titaniumnetwork-dev/Ultraviolet) as my bare client, as it was well-maintained and had a large userbase. For the webpage, I used a heavily-modified version of [Ultraviolet-Static](https://github.com/titaniumnetwork-dev/Ultraviolet-Static), as it was written in vanilla HTML.

## Hosting

For hosting the website, I decided to use Cloudflare Pages, as hosting both backend and frontend on Cloudflare's edge network would reduce the chance of the frontend being blocked via IP address (versus GitHub Pages, which was my host of choice)

## What next?

I plan on adding search suggestions and overall improving user experience.
