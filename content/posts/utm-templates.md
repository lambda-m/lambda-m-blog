+++
date = '2024-10-23T21:47:50+02:00'
draft = false
title = 'UTM Templates'
tags = ['UTM', 'HomeLab']
+++

A few notes on creating VM templates for use in UTM with UTMultiply, my script to quickly clone one or more VMs in UTM on macOS for homelabbing.

## SSH Keys

Ubuntu installer provides a way to import your public keys from github during setup. If you don;t have or want to use that, you'll need to copy them over manually. Once basic setup has finished, we need to login once via the UTM console to check the IP address (`ip a`) so we know where to SSH as the hostname of the guest is not yet known to the host.

```zsh
~ % ssh-copy-id -i .ssh/id_ecdsa.pub maarten@192.168.64.6
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: ".ssh/id_ecdsa.pub"
The authenticity of host '192.168.64.6 (192.168.64.6)' can't be established.
ED25519 key fingerprint is SHA256:pe6jMxXsHQh9R6itLYL/Kk9odDhj4BWlTpLUOs7plSA.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
maarten@192.168.64.6's password:

Number of key(s) added:        1

Now try logging into the machine, with:   "ssh 'maarten@192.168.64.6'"
and check to make sure that only the key(s) you wanted were added.
```
Now we can login using our SSH key:
```zsh
~ % ssh 'maarten@192.168.64.6'
Linux template-debian 6.1.0-26-arm64 #1 SMP Debian 6.1.112-1 (2024-09-30) aarch64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Oct 23 22:00:43 2024
maarten@template-debian:~$
```

## Disable IPv6

Since I am not using it, and it can interfere with the script parsing the IP address from `utmctl` I disable IPv6 entirely on the templates. For Debian / Ubuntu modify `/etc/sysctl.conf` and add the line `net.ipv6.conf.all.disable_ipv6 = 1`

## Passwordless sudo

Syntax for the sudoers file, not very secure, but highly convenient for lab setups:

`%sudo   ALL=(ALL) NOPASSWD:ALL`

Make sure to check if the user is in the sudo group:

```console
$ groups maarten
maarten : maarten cdrom floppy sudo audio dip video plugdev users netdev
```

## Software Packages

For UTMultiply to work as intended, it is necessary to make sure `qemu-guest-agent` is installed in the template, to allow `utmctl` to figure out which IP address the VM has received. If you want your machines to be reachable by name over your local LAN, you can install `avahi-daemon`. This enables [Multicast DNS](https://en.wikipedia.org/wiki/Multicast_DNS) (mDNS) and makes your machine reachable as `<hostname>.local`.

Pending availability for your distribution, but these are available in most comon distros.