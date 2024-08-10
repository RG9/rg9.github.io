---
title: "Three ways to configure HTTP Proxy in Playwright"
categories: [Code deep dives]
tags: [playwright, browserless, java]
---

TL;DR

In fact, in Playwright `v.1.31.1` there are **three** ways to configure an HTTP proxy:
* `chromium.launch`
* `browser.newContext`
* `page.setExtraHTTPHeaders`, but works only on Firefox

---

Today I was struggling with proxy setup for Browserless ...

[Official docs](https://www.browserless.io/docs/using-a-proxy) says:

```md
Both browserless, and Chrome itself, support the usage of external proxies. In order to fully utilize a 3rd-party proxy you'll need to do two things:

-   Specify the address of where the proxy is with the `--proxy-server` switch.
-   Optionally, you'll also need to send in your username and password if the proxy is authenticated.
```

Ok, so I need to set `--proxy-server`. I was using a Docker image to run Browserless locally. I discovered that there is an environment variable called `PROXY_URL`:
```sh
podman run --rm -p 3000:3000 -e "PROXY_URL=my-proxy-url" browserless/chrome:latest
```

Fine, but how do I pass on the username and password?
Docs say there are two methods:
```md
## Using username and password
### Method 1: page.authenticate
### Method 2: page.setExtraHTTPHeaders
```

I'm using `playwright-java` and there is no method `page.authenticate`, so `page.setExtraHTTPHeaders` left, I thought.

```java
page.setExtraHTTPHeaders(Map.of("Proxy-Authorization", "Basic "  
   + new String(Base64.getEncoder().encode("username:password".getBytes()))));
```

Unfortunately, it does not work. I received the following strange error: `net::ERR_INVALID_ARGUMENT`. Besides there was no errors in container's logs. What's the hack?

Google's top two results for `playwright Proxy-Authorization`:
- [node.js - How do I authenticate a proxy in playwright - Stack Overflow](https://stackoverflow.com/questions/67478486/how-do-i-authenticate-a-proxy-in-playwright)
- [Proxy-Authorization header not working in Chromium · Issue #443 · microsoft/playwright-python · GitHub](https://github.com/microsoft/playwright-python/issues/443)

I quickly checked the first link. The answer says that there are **two** ways. I didn't read further, because I thought that I know these ways - one is method I'm struggling with and second `chromium.launch`, right? :) *Later it turned out that I was wrong!*

The second link looks like what I need. However issue was closed without any conclusion. The alternative shown, was one that I already knew:
```javascript
browser = await playwright.chromium.launch(
    proxy={"server": "localhost:8080", "username": "user", "password": ""},
)
```

The case was that I didn't want to `lanuch` browser, but rather `connect` to one created inside container.

I dug even deeper and found another similar issue in Playwright project:
[# [Feature] Directly set Proxy-Authorization in Chrome without configuring any other proxy settings](https://github.com/microsoft/playwright/issues/11967)
> The problem (in Chrome/Chromium anyway) is that `Proxy-Authorization` may not be set explicitly via `extraHTTPHeaders`. If you do this and then try to navigate to any page Chrome will give you the following error: `net::ERR_INVALID_ARGUMENT`.

I was lost, considering changing my approach...

Fortunately, my teammate assisted me by showing some examples of where I found a missing piece:
```java
Browser.NewContextOptions options = new Browser.NewContextOptions()  
   .setProxy(new Proxy("my-proxy-url")  
      .setUsername("username")  
      .setPassword("password"));  
BrowserContext context = browser.newContext(options);
```
Ta da! As simple as that! 😃

I checked the [playwright docs](https://playwright.dev/docs/network#http-proxy) and it appears to be possible:
```
Proxy can be either set globally for the entire browser, or for each browser context individually.
```

Unluckily there is no example, so it's hard to spot.

I also double-checked [node.js - How do I authenticate a proxy in playwright - Stack Overflow](https://stackoverflow.com/questions/67478486/how-do-i-authenticate-a-proxy-in-playwright) and configuring proxy via `browser.newContext` was actually there, but I missed it (confused by Browserless docs?):
```javascript
const browser = await chromium.launch({
    proxy: { server: 'per-context' }
});
const context = await browser.newContext({
    proxy: { server: 'http://myproxy.com:3128' }
})
```

Getting to the point: 
*Festina lente* ("hurry slowly") and read comprehension!

