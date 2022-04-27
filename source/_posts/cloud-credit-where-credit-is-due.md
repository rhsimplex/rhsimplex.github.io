---
title: The Cloud Credit Trap
date: 2022-04-26 15:07:27
tags: [ml, mlops, aws]
---
I'm a big fan of [PyTorch Lightning](https://github.com/PyTorchLightning/pytorch-lightning) and use it [whenever I can](https://github.com/bayer-science-for-a-better-life/molnet-geometric-lightning). So I was excited to hear the Lightning creators were launching their own MLOps platform [GridAI](grid.ai). Unfortunately, through no fault of its creators, I will probably never use it.

## The credit trap

If I'm working at a big company and need MLOps, it's unlikely that I'll get permission to sign up for GridAI. I'll probably be forced to use SageMaker or VertexAI, the AWS and GCP equivalents respectively. Fair enough -- the company is big and established and so are the cloud providers they're already using.

What's counter-intuitive is that I'll probably get the same outcome at a startup. Why? The cloud credit trap: that well-known Faustian lock-in every startup CTO will take to get their company $50-100k of free cloud stuff. Why on earth would they authorize me to start spending hundreds or thousands of dollars on GridAI when I could be spending zeros of dollars on Sagemaker for the "same thing?" They've traded curiosity/risk for expedience, and are right to do so: I would do the same!

![](/images/maxresdefault.jpg)<center>*A skeptic tech lead listens intently to an AWS Startup Solutions Architect*</center>


## The irony

GridAI is [hosted on AWS](https://www.grid.ai/billing-rates/). But you can't use those AWS cloud credits on GridAI.

It's understandable. Imagine the nightmare of trying to apply AWS credits through billing from a third party. A cloud provider has no incentive to do that, especially if they want to push you towards _their_ managed offering. 

I'm not the first to point out this monopolistic quagmire. I just find this example -- a company giving you free money to get hooked on their product while also collecting cloud rent from their competitor's product -- particularly twisted.

It's hard to see a world where AWS or GCP would let cloud credits flow through to services built on their infrastructure. But one could imagine a less managed-service-oriented competitor explicitly encouraging this as a way to grow their overall customer base.

So who is the target user for something like GridAI? I want it to be me, but due to the cloud credit trap it's hard to imagine a professional situation where I'd ever use it.