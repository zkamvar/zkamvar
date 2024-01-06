---
title: Finally getting VPN to work again on Ubuntu
author: Zhian N. Kamvar
date: '2024-01-06'
slug: vpn
categories:
  - example
tags:
  - command line
  - ipv6
  - ipv4
  - vpn
  - nmcli
  - openVPN
subtitle: 'A journey in BASH'
excerpt: ''
draft: no 
series: ~
layout: single
---

For various reasons, I've been using a VPN for the last 6 years. The one I've
been using is [VyprVPN], but there are several others out there that do the job.
There was an app for Windows, Mac, iPhone, Android, and even a [CLI utility for
ubuntu](https://support.vyprvpn.com/hc/en-us/articles/360038091211-VyprVPN-CLI-for-Linux), and all of them appeared to work pretty well. Some time in the last year
or so, I was finding that **Google knew my geolocation** when I would use the
VPN on my Ubuntu machine, but not on any of the other devices, which meant that
my IP address was leaking.

It turns out that my internet provider started using IPv6 and that [most VPN
services do not support IPv6](https://restoreprivacy.com/vpn/best/ipv6/). The
best they can do is to disable IPv6, which is what the apps on my other devices
do. However, for some reason, the CLI utility that VyprVPN provided was not
doing this, so I needed to do three things:

1. Set up [OpenVPN for Ubuntu](https://support.vyprvpn.com/hc/en-us/articles/360037721812-VyprVPN-OpenVPN-Setup-for-Linux-Ubuntu-)
2. Download [the OpenVPN configuration files for the VyprVPN servers](https://support.vyprvpn.com/hc/en-us/articles/360037721812-VyprVPN-OpenVPN-Setup-for-Linux-Ubuntu-)
3. Write a script that would [disable IPv6][disable-ipv6] before turning on VPN.


I also want to check that my ipv6 is actually disabled by using <https://test-ipv6.com/> or
<https://ipleak.net/>.

[disable-ipv6]: https://support.vyprvpn.com/hc/en-us/articles/360038553552-How-do-I-disable-IPv6-on-Linux-

This post shows not only the solution, but also demonstrates BASH concepts of 
**arrays**, **control flow**, **functions**, and **process substitution**.

## Simple Solution

The bare bones solution would be to implement two functions that [toggle ipv6][disable-ipv6] and [connect to VPN via the command line][connect-vpn] via
[`nmcli`]

[connect-vpn]: https://askubuntu.com/a/57409/853075
[`nmcli`]: https://developer-old.gnome.org/NetworkManager/stable/nmcli-examples.html

1. `vpn_on`: disables ipv6 using `sysctl` and then turns on the VPN via 
   `nmcli con up id <VPN ID>`
2. `vpn_off` checks for an active VPN connection with `nmcli con show --active`,
   re-enables ipv6, and then turns off the VPN (if needed) via 
   `nmcli con down id <Active VPN ID>`

THis implementation looks like this and would default to a VPN ID called 
"Seattle":

```bash
vpn_on function () {
    sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1 \
    && sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1 \
    && sudo sysctl -w net.ipv6.conf.lo.disable_ipv6=1
    nmcli con up id "${1:-Seattle}"
    return 0
}

vpn_off function () {
    local active
    active=$(nmcli -f NAME,TYPE con show --active \
        | grep 'vpn\s*$' \
        | awk -F'  ' '{print $1}'
    )
    sudo sysctl -w net.ipv6.conf.all.disable_ipv6=0 \
    && sudo sysctl -w net.ipv6.conf.default.disable_ipv6=0 \
    && sudo sysctl -w net.ipv6.conf.lo.disable_ipv6=0
    if [ -z "${active}" ]
    then
      return 0
    else
      nmcli con down id "${active}"
      return 0
    fi
}
```

## Full Solution

Of course, the simple solution above requires me to know what VPN networks I can
connect to and it requires me to spell the connections correctly, so I needed
to write a few more functions. You can find what I came up with in [my `.bash_funs.sh` configuration file][znk-bash-profile]. 


### Getting a List of VPN networks available.

Getting a list of vpn networks is as straightforward as filtering the `vpn` type
from the connection list from [`nmcli`]:

```bash
vpn_list () {
    nmcli -f NAME,TYPE con show \
    | grep 'vpn\s*$' \
    | awk -F '  ' '{print $1}'
}
```

Note that some VPN network names will have spaces in the titles (such as South Korea), so that's why I need to set two spaces for `awk`.

This allows me to get a column of network names:

```bash
$ vpn_list | awk -F '  ' '{print NR") "$1}'
```
```
1) Seattle
2) Chicago
3) LA
4) South Korea
```

### Choosing a VPN network from a list interactively

Of course, the next challenge comes with the spelling aspect. I am lazy. I don't
want to have to type something multiple times, especially if I have to put it in
quotes if I don't have to. If I could choose from a list, that would make me so
happy. Luckily, I found out about [the BASH `select`
construct](https://linuxize.com/post/bash-select/), which would allow me to
select a specific item from an array:

```bash
local connections
connections=$( vpn_list )
select vpn in "${connections[@]}"
do
    echo ${vpn}
    return 0
done
```

However, it turns out that if there are any spaces in the array elements, BASH
will interpret those words as _separate elements_:


```
1) Seattle
2) Chicago
3) LA
4) South
5) Korea
```

Luckily, there is the [`readarray`] BASH builtin that helps with this. If we
read in the newline-separated array from [a process
substitution](https://vincebuffalo.com/blog/2013/08/08/using-names-pipes-and-process-substitution-in-bioinformatics.html),
then we can be sure that it will be read in as an array with each element
correctly placed:

[`readarray`]: https://www.baeldung.com/linux/reading-output-into-array

```bash
local connections
readarray -t connections < <( vpn_list )
select vpn in "${connections[@]}"
do
    echo ${vpn}
    return 0
done
```

```
1) Seattle
2) Chicago
3) LA
4) South Korea
```

### Manually supplying a VPN network name

There are times when I know the name of the VPN network and I don't want to 
select from a list. In these cases, I know that I'm a terrible speller, so I 
want to make sure that the VPN ID matches what is on my computer. It turns out
this [is a not an uncommon question](https://stackoverflow.com/q/3685970) and
[I lifted the answer directly from Stack Overflow](https://stackoverflow.com/a/8574392):

```bash
# https://stackoverflow.com/a/8574392
# 
# Returns success if the element exists in an array, a failure otherwise
#
# containsElement <element> <array>
function containsElement () {
  set +e
  local e match="$1"
  shift
  for e; do [[ "$e" == "$match" ]] && return 0; done
  return 1
}
```

This then allows me to then construct a `vpn_choose` function that will
optionally take a declared VPN network of choice. If the VPN network exists,
then the function exits early, and if not, I am provided with a list of options.

```bash
function vpn_choose () {
    local connections
    local declared="${1:-none}"
    # https://www.baeldung.com/linux/reading-output-into-array
    readarray -t connections < <( vpn_list )
  
    if containsElement "${declared}" "${connections[@]}"
    then
        echo "${declared}"
        return 0
    fi
  
    # https://linuxize.com/post/bash-select/
    select con in "${connections[@]}":
    do
        echo "${con}"
        return 0
    done
}
```

And this is all reflected in [the `vpn` function I wrote][znk-bash-profile]. It
has been something I've been meaning to implement for a while, but haven't 
really had the time to do so until now. I'm going to keep writing these little
blog posts as I go along so that I can get better at writing and find my voice.
If you've followed along this far, thank you!

[VyprVPN]: https://www.vyprvpn.com/
[znk-bash-profile]: https://github.com/zkamvar/config-files/blob/99265dff00ea6955abd4674b248a852f2761dd15/.bash_funs.sh#L101-L257
