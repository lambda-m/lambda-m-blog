+++
date = '2024-10-23T20:12:25+02:00'
draft = false
title = 'UTM Easy Clone & Deploy'
tags = ['UTM', 'HomeLab']
+++

Looking for a low friction way to spin up VMs in [UTM](https://mac.getutm.app/) for setting up homelab environments on an adequately specced MacBook. Sweet sweet (unified) memories.

`utmctl` provides some options for starting, sotpping and cloning machines

```
SUBCOMMANDS:
  list                    Enumerate all registered virtual machines.
  status                  Query the status of a virtual machine.
  start                   Start a virtual machine or resume a suspended virtual machine.
  suspend                 Suspend running a virtual machine to memory.
  stop                    Shuts down a running virtual machine.
  attach                  Redirect the serial input/output to this terminal.
  file                    Guest agent file operations.
  exec                    Execute an application on the guest.
  ip-address              List all IP addresses associated with network interfaces on the guest.
  clone                   Clone an existing virtual machine.
  delete                  Delete a virtual machine (there is no confirmation).
  usb                     USB device handling.
```

  This provides the basics, but some important settings cannot be changed from here. Most notably changing the MAC address of a cloned machine. In typical homelab scenarios for me, I want to use Shared Networking and after cloning a machine, perhaps multiple times, if they all have the same MAC address, the internal (QEMU?) DHCP server will assign the same IP address.

  I would like to be able to have a limited number of preconfigured VMs on the ready which I can clone using the least amount of effort possible. I should be able to give the VM a name, which should be its hostname, and add a few lines to my ssh config to be able to reach the machine using that name.

## Template VM prerequisites

- Up to date base install
- qemu-guest-agent
- sshd running with my public key
- Dynamic IP Configuration (DHCP)
- Additional software:
	- Backend services (ceph, incus, postgres, nginx, etc.)
	- GUI tools (vscode, sublime text, browser, HTTPS client for API testing, etc.)

Probably a handful of templates will need to be defined with a combination of additional softwares.

## 

I would ideally have a script that allows me to:

1. Select any of the Template VMs
2. Ask for a new hostname


It should result in a cloned VM, with a new name (in UTM) new MAC address and IPv4 address and it should allow me to immediately SSH into that host using a predefined config in ~/.ssh/config

e.g.

```console
> utmclone

Select Template:

	1. Ubuntu 22.04 LTS
	2. Debian 11 LTS
	3. Arch 2024.10.01

> please enter a number [Default: 1]: 2
> please enter a new hostname for Debian 11 LTS: my-new-clone

Cloning Debian 11 LTS... Done.
Randomizing MAC Address... Done.
Starting VM... Done.
Found IPv4 Address: 192.168.64.72
Setting SSH config:... Done.
Adding to /etc/hosts... Done.

You can now connect to your new machine "ssh my-new-clone"

>
```

Most of these things can be done using utmclone except as it stands, changing the MAC address. Luckily someone figured out how to do this using applescript! Although there are some limitations / caveats at first sight.

```bash
#!/bin/bash

VM_NAME="my_virtual_machine"

# Create a VM clone
utmctl clone "${VM_NAME}" --name "${VM_NAME}-clone"

# Randomize the MAC address
# (setting an empty string tells UTM to generate a random MAC) 
osascript <<END
tell application "UTM"
    set vm to virtual machine named "${VM_NAME}-clone"
    set config to configuration of vm
    set item 1 of network interfaces of config to {address:""}
    update configuration of vm with config
end tell
END
```

Source: [ServerFault](https://serverfault.com/a/1162322) - The UTM applescript cheatsheet can be found [here](https://docs.getutm.app/scripting/cheat-sheet/)

Perhaps it turns out to be easiest to do everything using applescript, as it seems it is able to 
