---
layout: post
title:  "Setup X2Go Client"
slug:   setup-client
date:   2018-10-10 12:58:29
categories: X2Go
tags: 
 - x2go
 - remote-access
 - rdp
 - ubuntu-18-04
 - windows
---

[server install guide]({{ site.baseurl }}{% link _posts/2018-10-11-setup-x2go-server.markdown %})

## Windows Client

There is a Windows client available to access remote linux servers.

The client can be downloaded from the [x2go site](https://wiki.x2go.org/doku.php/doc:installation:x2goclient).
Download `x2goclient-xxx-setup.exe` from the Windows link, and install like any other installer.

## Linux Client

First, install the X2Go PPA. While X2Go is available in the general repositories, this allows us to track the
newest patches and get additional add-ons not available in the general repository.

```
sudo apt-add-repository ppa:x2go/stable -y
```

Then, install the package:

```
sudo apt update && sudo apt install -y x2goclient
```

## Client Configuration - All OSes

Once the client is installed, configuration should be similar across OSes.

When the client opens, setup a new connection.

Set the following variables to connect to a serve:

 * Under the session tab:
   * Session name - This is an arbitrary string which will name the session
   * Host - An IP or DNS name with the server's hostname
   * Login - SSH Username on the host
   * SSH port: defaults to 22, modify as needed
   * If possible, use the RSA/DSA key. This is more secure than password connections. Specify the path
     to your private key.
   * Click the ghost icon if you'd like to set a new icon. This is purely aesthetic.
   * IMPORTANT: Ensure the session type is correct. More about this below.

 * Input/Output tab: 
   * Change the resolution to your desired size if you'd like.
	

## Selecting Session Type

The session type is an important setting - it decides what remote shell is connected to.

The first option for session type is picking a Display Environment (KDE, MATE, XFCE, etc). You must select
a DE which is already installed on the system, or the connection will fail. Selecting a DE will also force 
the user to connect to a background session, not the console output session.

The next option is using "Connect to local desktop". This is equivilent to a console connection, and
whatever is done on this view will also be visible to the system's monitors / VM console, if connected.

Finally, you can use "Single application" to run an X2Go session with just a single application, instead
of an entire remote window. This can be useful when you only ever need to run a remote application. If
this option is used, you must also specify the full path to the application. This does not always work well
on Windows (at least Windows 7 where I have tested it).

## Connecting

Once the session details are saved, you'll see a box on the right side of the client for your session.
Click the box to connect.
