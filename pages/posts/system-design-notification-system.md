---
title: 'System Design Look Back - A Notification System'
date: 2023/08/18
tag: system design
author: Andy
---

Years ago, when I designed a notification push feature for [bilibili.com](https://www.bilibili.com/), I used an architecture sharing one WebSocket connection in the browser to receive notification messages. From today's perspective, this design was petty stupid for not leveraging the Service Worker [notification API](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerRegistration/showNotification). But still, it is a system design case that I can learn from the past.

## Notification API

Basically, a notification is a message would pops up in the corner area of your OS, when something new you follow is updated like a YouTube channel you turn on the notification.

<img style="width: 200px" alt="notification" src="/images/notification.png" />

The Notification API was widely supported years ago, but lots of users complain about this feature since websites that don't have a deliberate strategy cause users to receive unwanted notifications.

Because of this, Google or the Chrome team published a new limitation in 2020 called "[quieter permission](https://blog.chromium.org/2020/01/introducing-quieter-permission-ui-for.html)". Bacically the browser itself determines if the site should be allowed to push notifications based on the worldwide historical permission accept rate and your habit of accepting notifications.

This "quieter permission" has become the default setting in the Chromium and it seems to work fine, nobody cares about these annoying notifications.

## System Design

Anyway, back to the system design topic. The architecture is simple. It is basically an architecture of the primary replica model.

1. There is a primary tab that creates a WebSocket connection to receive notification messages and push notifications.
2. And replica tabs detect the heartbeat from the primary by using the localStorage API.
3. Once the primary is closed and the heartbeat stops. Randomly choose one of the replica tabs to create a new WebSocket connection.

<iframe src="https://link.excalidraw.com/readonly/qR3ArTQvYlS256ADSRzh?darkMode=true" width="100%" height="600px" style="border: none;"></iframe>

Without a background running Service Worker, it is the optimal way to save bandwidth on the Websocket server since there is only one websocket connection all the time. This primary-replica model is widely used in the industry like Database write-read duty separation where there is always one instance receiving write operations while others own the duty of reading operations.

## Ending

Why would anyone want to be disturbed while they are working or in activities? I think in the future the notification API will be canceled or replaced and no one would care about it.
