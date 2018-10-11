---
layout: post
title:  "Setup X2Go Server"
slug:   setup-server
date:   2018-10-10 12:58:29
categories: X2Go
tags: 
 - x2go
 - remote-access
 - rdp
 - ubuntu-18-04
---

## What is X2Go?
X2Go is a linux tool similar to RDP for windows which allows remote access of a system with higher performance
than is possible with VNC.

VNC simply sends a video stream, but like RDP, X2Go does some client-side rendering of the window to improve
responsiveness. It also allows multiple users to start multiple sessions on a single system, similar to 
Microsoft Terminal Services.


## Add Repository
First, install the X2Go PPA. While X2Go is available in the general repositories, this allows us to track the
newest patches and get additional add-ons not available in the general repository.

```
sudo apt-add-repository ppa:x2go/stable -y
```

## Install X2Go Server package

With this command, x2go's server component should be installed.

```
sudo apt update && sudo apt install -y x2goserver x2goserver-xsession
```

Additionally, you may want the following addons, depending on your desktop environment:

```
# Mate
sudo apt install -y x2gomatebindings

# LXDE
sudo apt install -y x2golxdebindings
```

Note that X2Go server does not work well with Unity or Gnome. Mate is generally the easiest desktop
environment to use, because it is simple, lightweight, and well supported by x2go. KDE is also
fairly usable.

## Client Installation

X2Go has two main clients available, the main client simply called `x2goclient`, as well as a lighter
weight python client called `pyhoca`. However, pyhoca uses the configuration set in the main x2goclient,
so no matter which one you prefer, you must install `x2goclient`.

The client is installed on your desktop/laptop, not the server.

```
sudo apt update && sudo apt install -y x2goclient
```

Once this is installed, run `x2goclient` from your terminal, or find it in your applications menu. From
here, you can then set up a connection to a host.

TODO: Add additional setup steps for the client.
