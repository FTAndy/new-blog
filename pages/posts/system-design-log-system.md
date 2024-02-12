---
title: 'System Design Look Back - A Browser Monitor System'
date: 2024/01/26
tag: system design
author: Andy
---

Days ago, I interviewed for a position in Microsoft and the interviewer asked how to design a browser monitor system that makes sense for Bing.com. Basically, it is an open-ended system design question that can be delved into with lots of aspects.

In the domain of browser monitor systems, there are tools that are very popular in the industry and provide end-to-end services like [Sentry.io](https://sentry.io/welcome/), [LogRocket](https://logrocket.com/), [Datadog](https://www.datadoghq.com/), and [MicroSoft Clarity](https://clarity.microsoft.com/), etc.

Some critical product components include:

1. client SDK: event; performance; error; replay; route;
2. big data process storage: Kafka; ClickHouse; PostgreSQL
3. big data retrieve: session replay; product heatmap; error tracking; performance waterfall; customize SQL Graph
4. alert and monitor: data threshold; third-party integration; cron job

The monitor system is complex and valuable for products especially in Big companies since this type of data represents your product and not every company wants to share its secret with Sentry.io. Based on this, instead of using a public service company like Bytedance rather hire a group of engineers at the cost of billions of dollars to fork and develop the open-source version of Sentry.io.

## Browser Monitor SDK

Sentry.io is the leader of this domain, the [source code](https://github.com/getsentry/sentry-javascript) of the SDK is necessary to be researched.

There are 2 core components in Sentry Javascript browser SDK:

1. [Tracing](https://github.com/getsentry/sentry-javascript/blob/f47d11f93b3f3957b1899a48f0b569afaf2a9d81/packages/tracing-internal/src/browser/browsertracing.ts)

- performance metrics: LCP, FID, CLS, FP, FCP, RTT, longtask, etc.
- network: fetch, XHR, resource
- event: click, scroll, hover, navigation, visibility, exit, etc.
- error: JS error, network error, crash.
- data assemble: aggregation, time threshold, config

2. [Intrgration](https://docs.sentry.io/platforms/javascript/configuration/integrations/)

- session replay
- custom feedback
- log capture
- etc

<iframe src="https://link.excalidraw.com/readonly/DOeBCQ4omH8byZtSF2Wq?darkMode=true" width="100%" height="500px" style="border: none;"></iframe>

The SDK codebase is easy enough that you can copy the majority of the codebase to rebuild a new monitor system. The key problem is from the backend on how to integrate the data into your business model and make it customized. Maybe you have a complex e-commerce system, medical system, CMS system, etc. The back monitor system is completely isolated from your business backend, and you might need to hire more engineers to handle your requirements like Bytedance.

## End

The value of a customized monitor system is critical for a serious business. For a start-up, shipping a new feature ASAP is the first consideration, and using the industry standard is the optimal choice.

Building a such system is fun for engineers since there are so many domains that can be delved into and improve efficiency and save resources aside from growing your actual business.
