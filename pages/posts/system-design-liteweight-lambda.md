---
title: 'System Design Look Back - Serverless and Sandbox'
date: 2023/06/29
tag: system design
author: Andy
---

How to design a Serverless platform?

No, you do need to. (Doge)

The concept of Serverless became popular when Amazon published Lambda as the industry milestone back in 2014. It is a critical technique as the basic Infra of Cloud Services like AWS, Azure, and GCP. Old VM companies like Linode struggle in this serverless trend, either hard to make a long-term profit or end up being [acquired](https://www.akamai.com/newsroom/press-release/akamai-completes-acquisition-of-linode) by other companies since no Start-Up want to quickly verify their new business while managing a VM and hiring a DevOps engineer.

Everyone wants to build a Serverless platform somehow in big tech companies. In Alibaba and Bytedance, there are several teams working on how to use Node.js as the process-level isolation to build Serverless. And all of them ended up dead since it is hard to verify the value in this domain.

Nowadays, the high competition in this domain really makes creating a new product way easier than 5 or 10 years ago. It is petty funny that AWS Lambda and Cloudflare Worker have become the fundamental Infra for other new Cloud Services. Like Vercel Function and Netlify Function, both use AWS Lambda and Cloudflare Worker as their underlying Infra.

But there is still a space to discuss the need to build your own Serverless platform if you have the requirement for your user to execute their hostile code in your server and not be hijacked by Big Tech companies.

## Process Level Sandbox in Javascript

There is no need to talk about VM Sandbox since Docker and K8s dominate it.

In the domain of process-level Sandbox, there are several Sandbox libs that support exec code in a process level and keep the host safe. But it is still a greenfield even Cloudflare [opensource](https://github.com/cloudflare/workerd) their dependent Sandbox runtime lib.

There are some crucial components in process-level Sandbox:

- security: how to keep each code not breaking the host?
- isolation: how to isolate the effect of each process in runtime?
- resource limitation: how much memory and bandwidth can be used?
- module management: which module should be provided to users?
- scalability: how to manage the process and scalable them among different hosts?

and the other components around it:

- code storage/management: where does the code be stored and should it be encrypted?
- API Gateway: which network and subnetwork can be accessed?
- data transmission: which data serialization and network protocol should be used?

It is a complex system involved with lots of classic CS topics and to build it from scratch is merely impossible and non-profitable.

## Architechture

This [article](https://blog.cloudflare.com/mitigating-spectre-and-other-security-threats-the-cloudflare-workers-security-model/) from Cloudflare illustrates the basic architecture of a whole life cycle in process-level Sandbox.

<img style="height: 500px" alt="Architechture" src="http://blog.cloudflare.com/content/images/2020/07/Workers-architecture.svg" />

1. API triggers the exec of function, maybe it is from an inbound HTTP proxy
2. pass the handler to the HTTP server (the main process)
3. the main process manages a Sandbox process pool that allocates a bunch of V8 waiting to exec code
4. schedule the exec event with code to one of its Sandbox process
5. the Sandbox process uses a V8 to compile and exec code and return results to the main process
6. respond with the result to the outbound HTTP proxy and to the client

Cloudflare didn't open source all components they depend on, the [workerd](https://github.com/cloudflare/workerd?tab=readme-ov-file) lib is a tool to compile and exec Javascript code and offer isolation. It is just the runtime part of the architecture.

There are still other options in the community not complicate. Like [isolated-vm](https://github.com/laverdet/isolated-vm), [vm2](https://github.com/patriksimek/vm2), and [v8-sandbox](https://github.com/fulcrumapp/v8-sandbox). They are dedicated to different purposes in executing Javascript code on the server side and have their own trade-off.

- If you want to just exec sync Javascript code, [isolated-vm](https://github.com/laverdet/isolated-vm) is a good option.
- If you want a simple and easy way to manage a new Node.js env, [vm2](https://github.com/patriksimek/vm2) is a good option.
- If you want to use the Cloudflare Javascript standard and make your sandbox lightweight enough, [workerd](https://github.com/cloudflare/workerd?tab=readme-ov-file) is a good option.

## Ending

To wrap it up. The future of Serverless and Sandbox is bright. The feeling of using Cloudflare is like back in the old days when the Open Web concept was created. You can host your blog on your physical machine at your house. And everyone can access your content in your house. Now you handle your blog to Cloudflare in every edge node in worldwide through a code snippet. And no government can block your content since they can't block every node on Earth. A variation of the Open Web!
