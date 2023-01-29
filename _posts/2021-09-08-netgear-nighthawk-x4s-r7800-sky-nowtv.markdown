---
layout: post
title: Using the Netgear Nighthawk X4S R7800 with NowTV/Sky broadband
date: 2021-09-08T16:30:00.000+00:00
categories: routers netgear nowtv sky broadband
published: true
cover: "/assets/images/wopr.jpg"

---
Unlike our previous broadband supplier (Plusnet), NowTV didn't have an option in signup to decline their own router. Not wanting to add to our collection of unused or otherwise obsolete communications equipment, I gave them a call. The conversation, though not unpleasant, was nevertheless unsatisfactory in outcome. "You have to use our router, as stated in the contract" being roughly the gist. Whereas other contracts we've had have been keen to promote the use of their own supplied kit (understandably, as it's going to be easier to provide support this way), they have, to a one, provided enough "at your own risk" instructions for how to use your own. Not so NowTV - their router is supplied preconfigured with no ready access to any of the connection details.

TL;DR skip to [the solution](#solution)

The broadband switched over mid week, so for the sake of keeping everyone working we just used [the supplied router](https://www.cable.co.uk/broadband/providers/now-broadband/routers/) until the weekend. Credit where it's due, this little box did not provide the horror show that some of the NowTV broadband forum posters had suggested - I found the range and speed acceptable and it is capable of dual band. The admin configuration pages are logically presented (almost as if actually designed for ease use rather than as an objective exercise in obfuscation). It does lack the ability to provide a guest network WAP however, which is a major dealbreaker as I need to be able to share my WAN connection without exposing my LAN connection to guest users<sup>1</sup>.

Come Saturday and I'm settling in for an afternoon of no craic. A half hour duck-ducking turns up:

* "You have to connect with MER"
* "You need a router that supports MPoA"
* "You need a router that supports Option 61"

Now, I've neither the desire nor mental capability to become an expert in something I'll need to know about once every couple of years at most, so I've no shame admitting to having no idea what any of this means. Searches for "Netgear MPoA" and "Netgear MER" yielded nothing useful, with some forum posts going so far to suggest that flashing the Nighthawk to use non Netgear firmware would be necessary. However a search for "Netgear option 61" turned up [this Netgear KB article](https://kb.netgear.com/000062426/Which-NETGEAR-routers-support-DHCP-option-60-and-61), which suggested that the stock firmware was up to task. Here's my takeaway:

* The terms "MER" and "MPoA" are, in this context at least, irrelevant.
* Out of the connection protocols Netgear routers support, "PPPoE" is the one to use.
* NowTV requires a username and password, but this needs to be sent as a DHCP option (Client Identifier String aka "Option 61") and not as PPPoE params.
* Netgear firmware is capable of this, there is no need to flash to anything else<sup>2</sup>

### <a id="solution"></a>The solution

#### 1. Get the client identifier string from the NowTV router

The client identifier is a string that looks like this:

    1a2b3c4f5a30@nowtv|abcd1234

which is made up from: `[12 hex digits]@nowtv|[8 char password]`<sup>3</sup>

> Note: One NowTV forum poster suggested it was possible to construct this string using the WAN mac address (which can be found in the admin configuration pages) with any 8 chars for a password - I didn't try this so can't verify, but it may be worth trying. Also, the first part of my own client identifier string (obtained with Wireshark) is formed of the router's WAN mac address **except for the last digit** (i.e. WAN MAC ends with 32 but my id string ends with 30. The LAN MAC is the same, but ends with 31)

NowTV won't give you this string, and it's not available to view in the admin configuration pages, but it's fairly easy to pull by examining the network traffic with [Wireshark](https://www.wireshark.org). See [this article](https://www.georgebuckingham.com/sky-fibre-router-vdsl-password/) for step by step instructions (with pictures!) - essentially:

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

#### 2. Configure the Nighthawk router

Disconnect the NowTV router and connect up the Netgear router and modem.

* Update to [the minimum required firmware version](https://kb.netgear.com/000062426/Which-NETGEAR-routers-support-DHCP-option-60-and-61) ([R7800 downloads are here](https://www.netgear.com/support/product/R7800.aspx#download). Note that if the version difference is large it may be necessary to increment in smaller steps - see end of footnote<sup>2</sup>)
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

<sup>1</sup> The requirement to share WAN but not LAN also excludes the option to keep the NowTV router and just use the Nighthawk in AP mode as the LAN connections need to be open to allow the WAN connection to be shared across both routers.

<sup>2</sup> Though flashing to ww-drt firmware is quite simple, getting back was a bit of a challenge - I found that every version of the Netgear firmware I uploaded (including both the earliest [listed here](https://www.netgear.com/support/product/R7800.aspx#download) and what Netgear calls the [initial release](https://kb.netgear.com/29753/D7800-Firmware-Version-1-0-0-38-Initial-Release)) simply resulted in the ww-drt firmware resetting itself. The ww-drt firmware upgrade process apparently does not support .img files, only .bin files. After following many dead links to a file called `ddwrt-to-netgear-fw-R7800.bin`, I finally tracked down a copy in [this Netgear forum thread](https://community.netgear.com/t5/Nighthawk-WiFi-Routers/R7800-revert-firmware-from-ddwrt/td-p/1862938) (many thanks to forum user `@jandockx` - I have stashed a copy in [this github repo](https://github.com/kierandenshi/netgear-r7800) in case that link goes down). This update will put you back on Netgear firmware version `1.0.0.40`. I found it was not possible to jump up straight to the latest version - after uploading the .img file I'd get a message (something like) "this firmware is incompatible .." but I managed to get there by making small steps - first to `1.0.1.28`, then `1.0.2.04`, and then incrementally through the revisions by first digit (i.e. `1.0.2.1*` to any `1.0.2.2*` to any `1.0.2.3*`) until making it back to the current version.

<sup>3</sup> Thats a pipe character (shift + backslash on an Apple keyboard) preceding the password.