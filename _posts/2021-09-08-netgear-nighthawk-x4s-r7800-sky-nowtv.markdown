---
layout: post
title: Using the Netgear Nighthawk X4S R7800 with NowTV/Sky broadband
date: 2021-09-08T16:30:00.000+00:00
categories: routers netgear nowtv sky broadband
published: true

---
Something that should be easy, just made difficult with deliberate obfuscation on the part of Sky. This should work for any router that has support for "DHCP option 60 and 61" - for some Netgear routers a firmware update may be required ([this Netgear KB article](https://kb.netgear.com/000062426/Which-NETGEAR-routers-support-DHCP-option-60-and-61)).

### Solution
NowTV requires a username and password, but this needs to be sent as a DHCP option (Client Identifier String aka "Option 61") and not as PPPoE params.
A Client Identifier String looks like this:

    1a2b3c4f5a30@nowtv|abcd1234

which is made up from: `[12 hex digits]@nowtv|[8 char password]`

NowTV won't give you this string, and it's not available to view in the supplied router's admin configuration pages, but it's fairly easy to pull by examining the network traffic with [Wireshark](https://www.wireshark.org). See [this article](https://www.georgebuckingham.com/sky-fibre-router-vdsl-password/) for step by step instructions (with pictures!) - essentially:

* Turn off the wireless network on the NowTV router and connect to it with an RJ45 cable (this is optional, but doing it will limit network traffic to only the router and the computer you are using)
* Turn the router off
* Start capturing in Wireshark (make sure to select the Ethernet device as the source)
* Turn the router on
* Wait for it to make a successful internet connection
* Stop capturing in Wireshark
* Filter the captured data packets with `udp.port==67`
* Select one of the `DHCP Discover` packets and expand the `Bootstrap Protocol` line
* Right click on the line `Option: (61) Client identifier` and from the context menu select `Copy -> ..as printable text`
* Paste into a text editor or notepad note

You should see a string similar to `=1a2b3c4f5a30@nowtv|abcd1234`. Remove the `=` sign from the beginning and save it for the next step.


Disconnect the NowTV router and connect up the Netgear router and modem.

* Update to [the minimum required firmware version](https://kb.netgear.com/000062426/Which-NETGEAR-routers-support-DHCP-option-60-and-61) ([R7800 downloads are here](https://www.netgear.com/support/product/R7800.aspx#download). Note that if the version difference is large it may be necessary to increment in smaller steps - see footnote<sup>1</sup>)
* In the admin configuration, go to the `Advanced` tab
* Choose `Setup -> Internet Setup`
* Choose to enter details manually (do not use the setup wizard)
* For `Does your Internet connection require a login?` choose `No`
* `Account Name (If Required)` and `Domain Name (If Required)` can be left as they are
* For `Internet IP Address` choose `Get Dynamically from ISP`
* For `Domain Name Server (DNS) Address` choose `Get Automatically from ISP`
* For `Router MAC Address` choose `Use Default Address` (you can spoof the NowTV router with the `Use This MAC Address` but it wasn't necessary in my case)
* For `DHCP Option`, leave `Vendor Class Identifier String (Option 60):` blank, for `Client Identifier String (Option 61):` paste in the client id string obtained in step 1 (without the preceding `=` sign)
* Click on apply and it should just connect

### Notes


<sup>1</sup>I found it was not possible to jump up straight to the latest firmware version - after uploading the `.img` file I'd get a message (something like) "this firmware is incompatible .." BUT I managed to get there by making small steps - first to `1.0.1.28`, then `1.0.2.04`, and then incrementally through the revisions by first digit (i.e. `1.0.2.1*` to any `1.0.2.2*` to any `1.0.2.3*`) until making it up to the current version.

