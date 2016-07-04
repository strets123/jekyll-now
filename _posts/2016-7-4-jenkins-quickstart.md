---
layout: post
title: Jenkins Quickstart for travis users with AWS and Packer
---

The Jenkins continuous integration system is a fully featured, difficult to use intimidating behemoth. That said it is also widely adopted, lightweight to run and free apart from the hosting costs.

In this post I am going to describe how to get up and running quickly and use Jenkins to run a remote build script to generate a virtual machine template with Packer. This post can be useful in start-up situations where you are using a cloud-based bitbucket or other private git repository and you would like to test your installation script in different environments and then run your tests without paying more than $5-$10 per month for the tools and hosting.



