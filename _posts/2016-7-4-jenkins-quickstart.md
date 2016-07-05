---
layout: post
title: Jenkins Quickstart with AWS and Packer
---

The Jenkins continuous integration system is a fully featured, difficult to use intimidating behemoth. That said it is also widely adopted, lightweight to run and free apart from the hosting costs.

In this post I am going to describe how to get up and running quickly and use Jenkins to run a remote build script to generate a virtual machine template with Packer. This post can be useful in start-up situations where you are using a cloud-based bitbucket or other private git repository and you would like to test your installation script in different environments and then run your tests without paying more than $5-$10 per month for the tools and hosting.

We start off with a new bitbucket github repo and add two files for our test:

###mongodb.json

    {
      "provisioners": [
       
        {
          "type": "shell",
          "execute_command": "bash '{{.Path}}'",
          "scripts": [
                "dependencies.sh"
          ]
          
        }

      ],

      "post-processors": [
      
      ],

      "variables": {
        "access_key": "{{env `AWS_ACCESS_KEY`}}",
        "secret_key": "{{env `AWS_SECRET_KEY`}}"
      
      },

      "builders": [{
        "type": "amazon-ebs",
        "access_key": "{{user `access_key`}}",
        "secret_key": "{{user `secret_key`}}",
        "region": "us-west-2",
        "source_ami": "ami-9abea4fb",
        "instance_type": "t2.micro",
        "ssh_username": "ubuntu",
        "ami_name": "packer-example-latest"
      }

      ]
    }
    
###dependencies.sh

        set -e
	sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
	echo "deb http://repo.mongodb.org/apt/ubuntu "$(lsb_release -sc)"/mongodb-org/3.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.0.list
	sudo apt-get update
	sudo apt-get install -y mongodb-org

To set up Jenkins on AWS, use the [Bitnami Jenkins AMI](https://aws.amazon.com/marketplace/pp/B00NNZUF3Q/ref=srh_res_product_title?ie=UTF8&sr=0-2&qid=1467723585013) so that you can install it on the smallest VM possible. Following the installation process you will need to retrieve the generated password from the server logs, these can be found under instace actions in the EC2 console.

Once you have your CI server up and running it will be available on [box-url]/jenkins. Next it is time to configure some plugins for Jenkins. In my case I installed the Bitbucket plugin and the git scm plugin and the credentials binding plugin. This is done via the "Manage Jenkins" menu item.

Because we want to run our test installs etc. separately to this virtual machine on a standard Amazon ec2 machine or locally, then we use the packer tool to invoke our build script. To install packer, ssh to the bitnami box and run the following commands:

	cd ~
	wget https://releases.hashicorp.com/packer/0.10.1/
	unzip packer_0.10.1_linux_amd64.zip 
	sudo ln -s ~/packer /usr/bin/packer

Next you will need to create an access key and secret for the AWS API - this is so packer can call the api to create virtual machines and build your code. This is done via the AWS console.

Now we set up our build job and our packer file. Here I am going to assume we want to simply install MongoDB and have that install script run by packer from a shell script in a git repository. We are aiming for the following workflow:

1) User changes script in git repository
2) Hook on commit triggers a build on ~Jenkins
3) Jenkins passes AWS API credentials from its secrets database to environment variables in the execution environment.
4) Jenkins checks out the latest version of the code and runs a simple bash script to start a packer build.
5) Packer uses the AWS credentials to create an EC2 instance
6) Packer ssh's to the instance and copies across the approriate build script as listed in the packer file
7) Packer runs the build script and exits with an approriate status to show if the build works or not

First add some credentials by using the credentials app, do this by:

Click on credentials >> (global) >> Add credentials >>

Then select secret text add as many secrets as you want, labelling the secrets with IDs

We now need to create a build job on Jenkins to hold all of this configuration. Go back to the dashboard and click New Item >> Freestyle project. You will then come to a large web form where your build is to be configured.

Name your project and then select git repositories under source code management. Under build triggers check "Build when a change is pushed to BitBucket". Under Build Environment check "Use secret text(s) or file(s)" and add the environment variable names and the IDs to which those variables need to attach. These IDs correspond to the credentials you created earlier.

In the add build step dropdown click "Execute shell". You are now ready to add your build command as a shell script. This should obviously invoke other shell scripts so as not to end up putting all your deploy script in jenkins.

In our case we are building our mongodb AMI so the build command is:

    packer build mongodb.json
















