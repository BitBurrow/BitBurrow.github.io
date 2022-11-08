---
title: How to set up your own BitBurrow VPN server
---

## NOTE: THIS SOFTWARE DOES NOT EXIST YET

Everything below is proposed draft documentation. The software to do what is described is being developed and is not at all usable yet.

## Introduction

BitBurrow is a set of tools to help you set up and use a VPN server anywhere--at your parents' house, an office, or a friend's apartment. And you don't have to be good with computers. A BitBurrow VPN server will allow you to securely use the internet from anywhere in the world as if you were at your "VPN home".

## What you will need

1. A coupon code for a BitBurrow hub. If you do not have access to one, you can [set up your own hub](/how-to-set-up-a-bitburrow-hub) (this requires some computer experience) or ask your company or organization about doing this.
1. A Flint router (GL.iNet GL-AX1800), available from [GL.iNet](https://store.gl-inet.com/collections/smart-home-gateway-mesh-router/products/flint-gl-ax1800-dual-band-gigabit-wifi-6-openwrt-adguard-home) and [Amazon.com](https://amazon.com/dp/B09HBW45ZJ) and [WalMart](https://www.walmart.com/ip/-/187628398) and [other locations](https://www.gl-inet.com/where-to-buy/#europe).
1. Permission to set up a new router at your "VPN home" location.
1. If you plan to continue to use the existing router at your "VPN home", you will need the login password for this router.
1. An Android phone or tablet which can be used at your "VPN home".

## Why would you want to do this

1. **Restrictions**. When you are on a commercial VPN, websites often add additional security checks or outright block service. When this happens, rarely is it clear *why*. A VPN via a residential or business location is not subject to these restrictions.
1. **Cost**. Typically the cost of the VPN server hardware over its lifetime is much less than monthly VPN service fees. The existing internet connection is probably sufficient. The software is free.
1. **Trust**. Many commercial VPN providers log VPN connections and online activities.
1. **Access**. Banks, Netflix, and other sites will allow you to use their services as if you were at your "VPN home", even if you are physically in another country.
1. **Firewalls**. WiFi at airports and coffee shops often blocks known VPN servers, but a non-commercial VPN is less likely to be blocked.

## Why would you *not* want to do this

1. **Speed**. Internet speed over the VPN will be limited by the devices at both ends, and also by the upload speed of the "VPN home" internet connection, since this will be used for downloading via the VPN.
1. **Reliability**. If the power is out or a cable gets unplugged at the VPN server location, the VPN will be unavailable. If you depend on a VPN, have multiple options.
1. **Global servers**. Commercial VPN providers usually have VPN servers in dozens of countries. With BitBurrow, you are limited to the servers you set up or friends invite you to use.

## Getting started

1. Gather the items from the [What you will need](#what-you-will-need) section.
1. Install and run the BitBurrow app for Android on a device at the same location as the router. It is available from the [Google Play Store](https://play.google.com/store/apps/details?id=com.bitburrow.app) or [F-Droid](https://f-droid.org/en/packages/com.bitburrow.app/).

The BitBurrow app will walk you through the rest of the process. Here is a summary of that process.

1. Scan your QR-code for the BitBurrow coupon or enter the information manually.
1. You will be given a login key and be asked to write it down in a safe place.
1. Plug in the new router and connect the Android device to its WiFi.
1. If everything works as planned, BitBurrow will configure your router as a VPN server, prompting you if necessary. The process normally takes a few minutes.
1. You can use the same BitBurrow app to add, edit, and delete VPN client devices (phones, laptops, other routers, etc.) to use the internet through your "VPN home" location.

## Links

* [source code for the BitBurrow hub](https://github.com/BitBurrow/BitBurrow)
* [source code for the BitBurrow app](https://github.com/BitBurrow/BitBurrow/tree/main/app)
* [How to set up a BitBurrow hub](/how-to-set-up-a-bitburrow-hub)
* [source code for *this* website](https://github.com/BitBurrow/BitBurrow.github.io)
