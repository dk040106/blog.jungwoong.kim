---
layout: article
title: Making an automatic wifi connector (Linux)
tags:
  - Linux
  - Wifi
  - Python
  - English
key: wifi-auto-python-en
---

## Intro

I was making an automatic data sending mechanism for my school project. After reading the digit from elevator screen, Jetson Nano should send the data to Firebase Firestore. It worked when I run the code, but on the next day it stopped. I wondered why the code terminates, and found out that the wifi of our school turns off from 2 a.m to 6 a.m. Loss of internet connection stopped when it tried to connect to Firebase, and it terminated the entire code. So, I had to write a code that detects the internet connection and automatically connects back after some amount of time.

## Network-Manager

There were some ways to manage internet connection through, and I found NMCli ([Network-Manager document by Ubuntu](https://ubuntu.com/core/docs/networkmanager)) the best fit for our project. Below is a description of Network-Manager by Ubuntu.

NetworkManager is a system network service that manages your network devices and connections and attempts to keep network connectivity active when available. It manages Ethernet, WiFi, mobile broadband (WWAN) and PPPoE devices while also providing VPN integration with a variety of different VPN services.
{:.info}

###### Installation can be done with script below.
```bash
sudo apt install network-manager
```

###### Initial connection to a wifi network
```bash
nmcli d wifi connect <wifi-name> password <wifi-password>
```

###### Connection and disconnection
```bash
// connect
nmcli con up <wifi-name>

// disconnect
nmcli con down <wifi-name>
```

When you make another connection to a same network with ```nmcli d wifi connect```, it makes indices like ```<wifi-name>-1, <another-wifi-name>-2```. To reconnect to known network, use ```nmcli con up```.
{:.info}

## Implementing NMCli to Python

The original code that reads and sends data to Firebase was wrote in Python, and I wanted to make the code simple as possible.
