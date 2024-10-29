---
tags: Writeup
image: /img/bfm-fake-quote.png
---

# How I hijacked Arc Search
Hello! In this article, I’d like to go over how I was able to make an Arc Browse For Me result link which could display whatever content I wanted.

> In this article, I’ll refer to The Browser Company of New York as “BCNY”, and Browse For Me as “BFM”.

## The goal
Make a shared BFM link display whatever content I want.

## What is Arc Search?
Arc Search is the mobile companion app to BCNY’s popular desktop browser, Arc. Arc Search’s claim to fame is Browse For Me, an AI-powered search engine, similar to Perplexity or SearchGPT. In the Arc Search app, you can press the share button in a BFM page, which generates a link you can share with your friends (i.e. https://search.arc.net/UYwUAKlR1Qm5fWDYuzPO).

## Backstory
Recently while I was using BFM on Arc Search, I began to wonder why sharing the page took a long time (~5-10 seconds). So, I fired up [Proxyman](https://proxyman.io/ios) on my iPhone to intercept the HTTP traffic going to and from my phone, and what I found was surprising.

## How sharing a Browse For Me result works
When you press the Share button on a BFM page in Arc Search, it fires a `POST` request to `https://search.arc.net/api/share`. However, in the body of request, I found that the entire contents of the search page were sent as well. Here’s how a typical request looks:

```http
POST /api/share HTTP/1.1
Content-Type: text/plain; charset=utf-8
Host: search.arc.net
Connection: close
Content-Length: 142321
```
```json
{
  "stream": [],
  "raw": "",
  "shouldRetain": false,
  "finished": true
}
```

The `raw` key can be ignored, as the server also seems to ignore it. The real magic is in the `stream` key, which is an array of JSON objects. This contains everything that is displayed on the frontend.

All objects in `stream` have the following keys in common:
- `title`: string
- `type`: string, can be one of the following:
  - `emojiList` - A list with emoji as bullet points
  - `sectionTitle` - A subheading
  - `userFeedback` - Asks for user feedback
  - `citation` - A citation. This will show a “Verified” tag next to the url, regardless if it was fake.
  - `pageTitle` - The title at the top of the page
- `isFinished`: boolean
- `isPlaceholder`: boolean
There are also other values in objects which differ depending on the `type`.

**Any value in the body can be changed without the server caring**, which means you can easily falsify results.

Once a link is created, the server sends a response like the following:

```http
HTTP/1.1 200 OK
Cache-Control: public, max-age=0, must-revalidate
Content-Type: text/plain;charset=UTF-8
Date: Mon, 28 Oct 2024 20:30:45 GMT
Server: Vercel
Strict-Transport-Security: max-age=63072000
Vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Url
X-Matched-Path: /api/share
X-Vercel-Cache: MISS
X-Vercel-Id: cle1::iad1::wd6z4-1730147445251-0db83fb9f76c
Connection: close
Transfer-Encoding: chunked
```
```json
{"shareID":"PNmICgV7lW4zJxmVDr71"}
```

`shareID` seems to be the link’s ID. I navigated to `https://search.arc.net/<id>`, and lo and behold, my content was there!

![BFM showing custom page on desktop](/img/bfm-desktop.jpeg)

It even works on the Arc Search app, when visiting the shared link:

![Arc Search on iOS showing custom page](/img/bfm-mobile.png)

## What are the risks?
The ability to create custom search results brings with it dire consequences, most alarmingly the ability to falsify search results and pass them off as legitimate. For example, my hijacked page includes a fake quote from https://arc.net, which displays a “Verified” tag next to it.

![Fake quote from arc.net](/img/bfm-fake-quote.png)

This creates a risk of misinformation being spread through Arc Search, while it appears to be a legitimate search result. For example, a search for “Who won the 2020 US election?” could be modified to state the winner was Kermit the Frog. The link could then be shared online, and convince some people of the fabricated result.

## Demo
You can see a live version of a fake search page [here](https://search.arc.net/PNmICgV7lW4zJxmVDr71), and an archived version is available [here](https://archive.is/D9MYb) in case BCNY takes it down or they go out of business (VC money doesn’t last forever). For research purposes, the exact request body I used to generate it can be found [here](https://rentry.org/sqpu4hwu).

## Fix and Conclusion
BCNY can fix this issue by generating a fixed link when a result is first generated and keep the process completely opaque from the client, similar to what competing services do already. But for now, this has been BomberFish, and I do declare; end broadcast.
