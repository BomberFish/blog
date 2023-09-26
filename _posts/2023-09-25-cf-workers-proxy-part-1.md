# Proxying on the Edge: Part 1

> This article is a part of an ongoing series on running web proxies using Cloudflare Workers.

Recently, I made a proxy called [Superposition](https://superposition.bomberfish.ca), running on Cloudflare's Workers serverless platform.

You can check out its source code [here](https://github.com/BomberFish/Superposition).

## How?

It turns out there's already a project for proxying requests using CF Workers, called [Workers-Proxy](https://github.com/aD4wn/Workers-Proxy). It works by making a request to a fixed URL, then returning the response to the user. The only issue is that it only works with a single domain, which is hard-coded into the program.

Superposition fixes this issue by checking for a `?url=` parameter in the requested URL. If the parameter is present, it proxies the provided URL. If not, it returns a landing page for you to input a URL.

Another feature of Superposition is that when it proxies a website, it also injects a button fixed to the top-left of the screen to go back to the landing page.

## Why?

The idea was simple: Since existing proxy websites rely on a central server which may or may not be close to the user, why not use an established global network such as Cloudflare to proxy the requests?

## What next?

I am currently working on a new proxy, dubbed **Infrared**, which is completely compliant with the [TompHTTP](https://tomp.app) specifications. Infrared aims to be more stable than Superposition, and includes more advanced features such as WebSocket proxying.
