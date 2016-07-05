---
layout: post
title: Jenkins Quickstart with AWS and Packer
---

The Jenkins continuous integration system is a fully featured, difficult to use intimidating behemoth. That said it is also widely adopted, lightweight to run and free apart from the hosting costs.

In this post I am going to describe how to get up and running quickly and use Jenkins to run a remote build script to generate a virtual machine template with Packer. This post can be useful in start-up situations where you are using a cloud-based bitbucket or other private git repository and you would like to test your installation script in different environments and then run your tests without paying more than $5-$10 per month for the tools and hosting.

To set up Jenkins on AWS, use the [Bitnami Jenkins AMI](https://aws.amazon.com/marketplace/pp/B00NNZUF3Q/ref=srh_res_product_title?ie=UTF8&sr=0-2&qid=1467723585013) so that you can install it on the smallest VM possible. Following the installation process you will need to retrieve the generated password from the server logs, these can be found under instace actions in the EC2 console.

Once you have your CI server up and running it will be available on [box-url]/jenkins. Next it is time to configure some plugins for Jenkins. In my case I installed the Bitbucket plugin and the git scm plugin. This is done via the "Manage Jenkins" menu item.

Because we want to run our test installs etc. separately to this virtual machine on a standard Amazon ec2 machine or locally, then we use the packer tool to invoke our build script. To install packer, ssh to the bitnami box and run the following commands:

	cd ~
	wget https://releases.hashicorp.com/packer/0.10.1/
	unzip packer_0.10.1_linux_amd64.zip 
	sudo ln -s ~/packer /usr/bin/packer

Next you will need to create an access key and secret for the AWS API - this is so packer can call the api to create virtual machines and build your code. This is done via the AWS console






