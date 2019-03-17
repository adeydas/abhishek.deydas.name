---
layout: blog
title:  "Easy Way to Protect Myself on Public Wifi Networks"
date:   2018-08-12 12:28:39 -0700
categories: tech
---
Public wifi has been a big part of my daily driver after moving to Vancouver. Every shopping mall, shop or restaurant seem to provide free wifi access to customers. There are even ISP’s who provide WiFi access through local vendors. All this is really good because it helps cut down on mobile data costs but a thing of concern is security. An [article by Symantec](https://ca.norton.com/internetsecurity-privacy-risks-of-public-wi-fi.html) lists a few risks of using public WiFi networks without the right security measures. A [recent survey by CNBC](https://www.cnbc.com/2016/06/28/most-people-unaware-of-the-risks-of-using-public-wi-fi.html) found that 60% of US public WiFi consumers weren’t aware of the security implications of using public WiFi networks.

For me, the challenge was to figure out a simple way to secure data transmission when connecting to public WiFi networks on my and my wife’s phone, where some of these networks will auto-connect since we frequent those places.

# Choice of tech
VPN is an obvious choice here but the choice comes with a couple of questions: (1) which VPN server to choose and (2) which protocol to use.

## 1. Which VPN server to choose

There are several free and paid VPN services out there and they come with their own pros and cons. If your usecase is to just tunnel to a VPN server, I would recommend a paid one like ExpressVPN. Free ones may track and sell your data and is probably not worth saving a few bucks. A second, more pertinent, concern in my case was access to Netflix and access to services internal to my home network. As you may know, Netflix is cutting down really hard on access via VPN services and most VPN services are not allowed to access Netflix anymore. Since I also wanted to access services within my home network like my Plex server, I decided to go with a VPN server within my home network.

## 2. Which protocol to use

OpenVPN is pretty standard and the protocol it uses is well regarded and because of those reasons, I decided to go with that on my first iteration. But there was a problem. It required manual intervention to connect to the server every time I logged into a public network. I wanted a seamless experience where I would be auto-connected to my VPN server every time I logged into an unknown/public WiFi network and automatically logged out when I switched to LTE or my home WiFi network. I will detail how I got that bit done in a later section. However, a side effect of that was dropping OpenVPN and choosing L2TP/IPSec as that was the only supported protocol by the tech I used.

# Architecture
![VPN architecture](https://s3.ca-central-1.amazonaws.com/static.abhis.ws/protect-wifi-network/vpn_architecture.png)
A few clarifying points on the architecture:

1. I do not have a static IP but I have a QNAP NAS with dynamic DNS setup. So I just used the same external URL as that of my NAS.
2. Services like Bonjour will not be available remotely with this architecture because the VPN subnet will be different from that of the access point/switch and Bonjour doesn’t work out of the box across subnets.
3. To access Plex servers, you will still need to allow remote access but disable Upnp, so Plex doesn’t open ports randomly. This is because, like (2), VPN users will be on a different subnet and Plex will drop such connections if remote access is not allowed.

# Setting up the VPN server
1. I used this docker image to set up a L2TP/IPSec server with a shared secret.
2. Once the server was up and running, I added a couple of port forwarding rules for UDP ports 500 and 4500 on my router.
3. To allow inbound VPN connections, I also added some rules to my firewall. I am not putting my rules here because they will vary based on what you want.

A very easy way to test whether the whole workflow works is to try connect to the VPN server locally and then try connect using the public IP from a different public IP. For me, I had to switch to LTE on my phone and test the connection using my ISP’s public IP and then a dynamic DNS domain attached to it. Please note that connecting from a different WiFi network using the same public IP will not work.

# Autoconnect when on a public WiFi
Thomas Witt has a [wonderful blog post](https://thomas-witt.com/auto-connect-your-ios-device-to-a-vpn-when-joining-an-unknown-wifi-d1df8100c4ba) on how to go about it, so I will skip the details. However, I will add my VPN profile for iOS devices. If you have a simple usecase like me with one VPN server and one WiFi network, simple replace ‘CHANGE_ME’ to the appropriate value and you should be good to go.
````xml
	<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>PayloadContent</key>
  <array>        <dict>
      <key>UserDefinedName</key>
      <string>AbhiNita VPN profile</string>
      <key>PayloadDisplayName</key>
      <string>AbhiNita VPN profile</string>
      <key>PayloadIdentifier</key>
      <string>ws.abhis.abhinita.vpn</string>
      <key>PayloadUUID</key>
      <string>1D68A15E-BB76-9EA9-DF80-4C2C8E3E4859</string>
      <key>VPNType</key>
      <string>L2TP</string>
      <key>IPSec</key>
      <dict>
        <key>AuthenticationMethod</key>
        <string>SharedSecret</string>
        <key>LocalIdentifierType</key>
        <string>KeyID</string>
        <key>SharedSecret</key>
        <string>CHANGE_ME_TO_YOUR_SHARED_SECRET</string>
      </dict>
      <key>PPP</key>
      <dict>
        <key>AuthName</key>
        <string>CHANGE_ME_TO_YOUR_AUTH_USERNAME</string>
        <key>AuthPassword</key>
        <string>CHANGE_ME_TO_YOUR_AUTH_PASSWORD</string>
        <key>CommRemoteAddress</key>
        <string>CHANGE_ME_TO_YOUR_VPN_SERVER_ADDRESS</string>
      </dict>
      <key>OnDemandEnabled</key>
   <integer>1</integer>
   <key>OnDemandRules</key>
          <array>
        <dict>
          <key>InterfaceTypeMatch</key>
          <string>WiFi</string>
          <key>SSIDMatch</key>
          <array>
 <string>CHANGE_ME_TO_YOUR_HOME_WIFI_SSID</string>
           </array>
          <key>Action</key>
          <string>Disconnect</string>
        </dict>
        <dict>
          <key>InterfaceTypeMatch</key>
          <string>WiFi</string>
          <key>Action</key>
          <string>Connect</string>
        </dict>
        <dict>
          <!-- VPN Default state -->
          <key>Action</key>
          <string>Disconnect</string>
        </dict>
          </array>
      <key>OverridePrimary</key>
      <true/>
      <key>IPv4</key>
      <dict>
        <key>OverridePrimary</key>
        <integer>1</integer>
      </dict>
      <key>PayloadType</key>
      <string>com.apple.vpn.managed</string>
      <key>PayloadVersion</key>
      <integer>1</integer>
    </dict>
</array>
  <key>PayloadDisplayName</key>
  <string>VPN Configurations</string>
  <key>PayloadIdentifier</key>
  <string>06A84332-4F0F-BD88-CA71-3E63EA0E6D56</string>
  <key>PayloadRemovalDisallowed</key>
  <false/>
  <key>PayloadType</key>
  <string>Configuration</string>
  <key>PayloadUUID</key>
  <string>88D906BD-78F0-B549-4CD2-1E542DA82A0B</string>
  <key>PayloadVersion</key>
  <integer>1</integer>
</dict>
</plist>
````

This method, by no means, is fool proof but will give a certain level of protection while using public WiFi networks along with access to resources within your home network. There’s certainly ground for improvement like using more secure protocols, making the experience of logging into public WiFi networks with splash pages more seamless, etc but more on that later.