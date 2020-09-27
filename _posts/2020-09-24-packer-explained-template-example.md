---
layout: post
title: "Packer a Complete Guide with Example"
author: nandan
categories: [devops]
excerpt: "In this article you'll look at what is packer, advantages and how to use packer in your environment. Along with a working example."
image: https://miro.medium.com/max/700/1*GP_BvGp4DsBNxfjAaXWZ_Q.jpeg
tags: [packer, hashicorp, featured]
---

## What is Packer?

Packer is easy to use and automates the creation of any type of machine image. It embraces modern configuration management by encouraging you to use a framework such as Chef or Puppet to install and configure the software within your Packer-made images.

Through out this article I have marked direct hyperlinks to Packer official docs whereever necessary.

## Advantages of Using Packer

- **_Super fast infrastructure deployment:_** Packer images allow you to launch completely provisioned and configured machines in seconds, rather than several minutes or hours.

- **_Multi-provider portability:_** Because Packer creates identical images for multiple platforms, you can run production in AWS, staging/QA in a private cloud like OpenStack, and development in desktop virtualization solutions such as VMware or VirtualBox. Each environment is running an identical machine image, giving ultimate portability.

- **_Improved stability:_** Packer installs and configures all the software for a machine at the time the image is built. If there are bugs in these scripts, they’ll be caught early, rather than several minutes after a machine is launched.

- **_Greater testability:_** After a machine image is built, that machine image can be quickly launched and smoke tested to verify that things appear to be working.

## Key Packer Terminologies

The terminology is in alphabetical order for quick referencing.

- **Artifacts** are the results of a single build, and are usually a set of IDs or files to represent a machine image. Every builder produces a single artifact. As an example, in the case of the Amazon EC2 builder, the artifact is a set of AMI IDs (one per region). For the VMware builder, the artifact is a directory of files comprising the created virtual machine.

- **Builds** are a single task that eventually produces an image for a single platform. Multiple builds run in parallel. Example usage in a sentence: “The Packer build produced an AMI to run our web application.” Or: “Packer is running the builds now for VMware, AWS, and VirtualBox.”

- **Builders** are components of Packer that are able to create a machine image for a single platform. Builders read in some configuration and use that to run and generate a machine image. A builder is invoked as part of a build in order to create the actual resulting images. Example builders include VirtualBox, VMware, and Amazon EC2. Builders can be created and added to Packer in the form of plugins.

- **Commands** are sub-commands for the packer program that perform some job. An example command is “build”, which is invoked as packer build. Packer ships with a set of commands out of the box in order to define its command-line interface.

- **Post-processors** are components of Packer that take the result of a builder or another post-processor and process that to create a new artifact. Examples of post-processors are compress to compress artifacts, upload to upload artifacts, etc.

- **Provisioners** are components of Packer that install and configure software within a running machine prior to that machine being turned into a static image. They perform the major work of making the image contain useful software. Example provisioners include shell scripts, Chef, Puppet, etc.

- **Templates** are JSON files which define one or more builds by configuring the various components of Packer. Packer is able to read a template and use that information to create multiple machine images in parallel.

We’ll look more at Builders, Provisioners & Post-Processors later in this article.

## Installing Hashicorp Packer

Packer may be installed in the following ways by visiting the official packer downloads page: [https://www.packer.io/downloads.html](https://www.packer.io/downloads.html)

## Basic Ready to Use Template

<script src="https://gist.github.com/itsLucario/fcdc2fe790a4e63cb5b2c3974dba135a.js"></script>

When building, we’ll pass in aws_access_key and aws_secret_key as [user variables](https://www.packer.io/docs/templates/user-variables.html), keeping your secret keys out of the template.

## Environment variables

You can provide your credentials via the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY, environment variables, representing your AWS Access Key and AWS Secret Key, respectively. Note that setting your AWS credentials using either these environment variables will override the use of AWS_SHARED_CREDENTIALS_FILE and AWS_PROFILE. The AWS_DEFAULT_REGION and AWS_SESSION_TOKEN environment variables are also used, if applicable:

**Usage:**

export AWS_ACCESS_KEY_ID=”anaccesskey”

export AWS_SECRET_ACCESS_KEY=”asecretkey”

export AWS_DEFAULT_REGION=”us-west-2"

## Shared Credentials file

You can use an AWS credentials file to specify your credentials. The default location is `$HOME/.aws/credentials` on Linux and OS X, or `“%USERPROFILE%\.aws\credentials”` for Windows users. If we fail to detect credentials inline, or in the environment, Packer will check this location. You can optionally specify a different location in the configuration by setting the environment with the AWS_SHARED_CREDENTIALS_FILE variable.

The format for the credentials file is like so

    [default]
    aws_access_key_id=<your access key id>
    aws_secret_access_key=<your secret access key>

You may also configure the profile to use by setting the profile configuration option, or setting the AWS_PROFILE environment variable:

    {
      "profile": "customprofile",
      "region": "us-east-1",
      "type": "amazon-ebs"
    }

## What are Builders?

A builder is a component of Packer that is responsible for creating a machine and turning that machine into an image. The builders section contains an array of JSON objects configuring a specific _builder_.

In this case, we’re only configuring a single builder of type amazon-ebs. This is the Amazon EC2 AMI builder that ships with Packer. This builder builds an EBS-backed AMI by launching a source AMI, provisioning on top of that, and re-packaging it into a new AMI.

    "builders": [{
      "type": "amazon-ebs",
      "access_key": "{% raw %}{{user `aws_access_key`}}{% endraw %}",
      "secret_key": "{% raw %}{{user `aws_secret_key`}}{% endraw %}",
      "region": "us-east-1",
      "source_ami_filter": {
        "filters": {
          "virtualization-type": "hvm",
          "name": "ubuntu/images/*ubuntu-xenial-16.04-amd64-server-*",
          "root-device-type": "ebs"
        },
        "owners": ["099720109477"],
        "most_recent": **true
      **},
      "instance_type": "t2.micro",
      "ssh_username": "ubuntu",
      "ami_name": "packer-example {{timestamp}}"
    }]

**Required Properties for Builder:**

[access_key](https://www.packer.io/docs/builders/amazon-ebs.html#access_key) (string) — The access key used to communicate with AWS. [Learn how to set this](https://www.packer.io/docs/builders/amazon.html#specifying-amazon-credentials). This is not required if you are using use_vault_aws_engine for authentication instead.

[ami_name](https://www.packer.io/docs/builders/amazon-ebs.html#ami_name) (string) — The name of the resulting AMI that will appear when managing AMIs in the AWS console or via APIs. This must be unique. To help make this unique, use a function like timestamp (see [template engine](https://www.packer.io/docs/templates/engine.html) for more info).

[instance_type](https://www.packer.io/docs/builders/amazon-ebs.html#instance_type) (string) — The EC2 instance type to use while building the AMI, such as t2.small.

[region](https://www.packer.io/docs/builders/amazon-ebs.html#region) (string) — The name of the region, such as us-east-1, in which to launch the EC2 instance to create the AMI.

[secret_key](https://www.packer.io/docs/builders/amazon-ebs.html#secret_key) (string) — The secret key used to communicate with AWS. [Learn how to set this](https://www.packer.io/docs/builders/amazon.html#specifying-amazon-credentials). This is not required if you are using use_vault_aws_engine for authentication instead.

[source_ami](https://www.packer.io/docs/builders/amazon-ebs.html#source_ami) (string) — The initial AMI used as a base for the newly created machine. source_ami_filter may be used instead to populate this automatically.

**Optional:**

[source_ami_filter](https://www.packer.io/docs/builders/amazon-ebs.html#source_ami_filter) (object) — Filters used to populate the source_ami field.

This selects the most recent Ubuntu 16.04 HVM EBS AMI from Canonical. NOTE: This will fail unless exactly one AMI is returned. In the above example, most_recent will cause this to succeed by selecting the newest image.

**filters** (map of strings) — filters used to select a source_ami. NOTE: This will fail unless exactly one AMI is returned. Any filter described in the docs for [DescribeImages](http://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeImages.html) is valid.

**owners** (array of strings) — Filters the images by their owner. You may specify one or more AWS account IDs, “self” (which will use the account whose credentials you are using to run Packer), or an AWS owner alias: for example, amazon, aws-marketplace, or microsoft. This option is required for security reasons.

**most_recent **(boolean) — Selects the newest created image when true. This is most useful for selecting a daily distro build.

You may set this in place of source_ami or in conjunction with it. If you set this in conjunction with source_ami, the source_ami will be added to the filter. The provided source_ami must meet all of the filtering criteria provided in source_ami_filter; this pins the AMI returned by the filter, but will cause Packer to fail if the source_ami does not exist.

These are basic configuration criteria required for a basic image creation. You can find the detailed options which we can specify by going through particular builder configuration

The above shown are the configuration options which we can use for — **Amazon EBS instance [**https://www.packer.io/docs/builders/amazon-ebs.html](https://www.packer.io/docs/builders/amazon-ebs.html)

**Azure** requires some additional setup for creating ARM images will detail in further documentation

[https://www.packer.io/docs/builders/azure-setup.html](https://www.packer.io/docs/builders/azure-setup.html)

## What are Provisioners?

Provisioners use builtin and third-party software to install and configure the machine image after booting. Provisioners prepare the system for use, so common use cases for provisioners include:

- Installing packages

- Patching the kernel

- Creating users

- Downloading application code

Packer has support for huge set provisioners which we can use,

[Ansible Local](https://www.packer.io/docs/provisioners/ansible-local.html), [Ansible (Remote)](https://www.packer.io/docs/provisioners/ansible.html), [Breakpoint](https://www.packer.io/docs/provisioners/breakpoint.html), [Chef Client](https://www.packer.io/docs/provisioners/chef-client.html), [Chef Solo](https://www.packer.io/docs/provisioners/chef-solo.html), [Converge](https://www.packer.io/docs/provisioners/converge.html), [File](https://www.packer.io/docs/provisioners/file.html), [InSpec](https://www.packer.io/docs/provisioners/inspec.html), [PowerShell](https://www.packer.io/docs/provisioners/powershell.html) ,[Puppet Masterless](https://www.packer.io/docs/provisioners/puppet-masterless.html), [Puppet Server](https://www.packer.io/docs/provisioners/puppet-server.html), [Salt Masterless](https://www.packer.io/docs/provisioners/salt-masterless.html), [Shell](https://www.packer.io/docs/provisioners/shell.html), [Shell (Local)](https://www.packer.io/docs/provisioners/shell-local.html), [Windows Shell](https://www.packer.io/docs/provisioners/windows-shell.html), [Windows Restart](https://www.packer.io/docs/provisioners/windows-restart.html), [Custom](https://www.packer.io/docs/provisioners/custom.html)

Each provisioner will have set of options required to be set in order for them to work.

**Let’s Have Look at File Provisioner for Example**

The file Packer provisioner uploads files to machines built by Packer. The recommended usage of the file provisioner is to use it to upload files, and then use [shell provisioner](https://www.packer.io/docs/provisioners/shell.html) to move them to the proper place, set permissions, etc.

**NOTE**: You can only upload files to locations that the provisioning user (generally not root) has permission to access. Creating files in /tmp and using a shell provisioner to move them into the final location is the only way to upload files to root owned locations.

    {
      "type": "file",
      "source": "app.tar.gz",
      "destination": "/tmp/app.tar.gz"
    }

## What are Post-Processors?

Post-processors run after the image is built by the builder and provisioned by the provisioner(s). Post-processors are optional, and they can be used to upload artifacts, re-package, or more. For more information about post-processors, please choose an option from the sidebar.

## ­­­Running Packer Build

To build image use below commandm

    packer build example.json

Available commands are:

- **build:** build image(s) from template

- **console:** check that a template is valid

- **fix:** fixes templates from old versions of packer

- **inspect:** see components of a template

- **validate:** check that a template is valid

- **version:** Prints the Packer version

## A Complete Packer Template Example

We have looked into all the major packer terminlogies and how to use them. Let’s now create a new template, which involves all that we learned and try it out:

**A sample ready to go template example for AMI Ubuntu with MySQL**

This example combines variables, provisioners and builders in a single file. The below template is tested with AWS EC2.

<script src="https://gist.github.com/itsLucario/fc66af41ba4d8eb9da30f3337f8b23f5.js"></script>

As you can see in the above template JSON. we are using a shell provisioner which executes the mysql.sh script. Let’s create the shell script for mysql installation.

<script src="https://gist.github.com/itsLucario/b63c233160ecd30e9d610a6fe184e9ca.js"></script>

Now we have everything ready. Let’s do the build with template that we created above.

Run the below command to build an AWS native AMI image: (In the below command path is wrt the [github example](https://github.com/itsLucario/packer-casic-example))

    packer build ./templates/ubuntu-mysql.json

![packer build output](https://cdn-images-1.medium.com/max/2000/1*lymHwXrJ4WxHwiffG60O-A.png)

Once the build gets successful, an AMI will get created in the specified region in our case its us-east-1 . You can login to your AWS Console, Open the Amazon EC2 console at [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/) and In the navigation pane, choose **AMIs**.

![](https://cdn-images-1.medium.com/max/2938/1*dHdAAfNRL8xGXp6EdZ-zOg.png)

You can see in the above image an AMI got created which matches with the AMI ID show in our terminal. You can use the image to create an instance right away.

Take a look at the working example on my GitHub. Feel free to leave a comment and don’t forget to leave claps and follow me if you find this helpful.

GitHub Coding Example
[itsLucario/packer-basic-example](https://github.com/itsLucario/packer-basic-example)
