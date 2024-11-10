+++
date = '2024-10-27T08:28:19+01:00'
draft = false
title = 'UTMultiply'
tags = ['UTM', 'HomeLab']
+++

I have created an initial version of my UTM Cloning script available [here](https://github.com/lambda-m/UTMultiply). It makes a bunch of assumptions on usage and setp, and requires templates to be set up in a certain way. The script takes care of a number of things to quickly spin up one or more machines for a project / experiment / itch.

## In Action (Ubuntu)

```console
maarten@Maartens-MacBook-Pro UTMultiply % ./utmultiply.sh
Fetching list of Template VMs... [Done]

Select Template:
   1. Template - Debian 12.7.0
   2. Template - Ubuntu 22.04 LTS


Please enter a number [Default: 1]: 2
You selected: Template - Ubuntu 22.04 LTS
Please enter a new hostname for Template - Ubuntu 22.04 LTS: demo-clone
Hostname is valid: demo-clone
Cloning Template - Ubuntu 22.04 LTS as demo-clone...[Done]
Randomizing MAC address...[Done]
Starting VM...[Done]
Retrieving IP address...[Done]
Setting hostname on the VM...[Done]
Updating SSH config...[Done]

Summary:
  VM Name     : demo-clone
  IP Address  : 192.168.64.21
  SSH Command : ssh demo-clone
```

After which you can use SSH to connect, using the entry created in `~/.ssh/config`:

