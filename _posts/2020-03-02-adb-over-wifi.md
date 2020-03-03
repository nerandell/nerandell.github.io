--- 
layout: post 
title: "Use ADB over WiFi"
date: 2020-03-02 14:31:18
summary: Using adb over WiFi to make debugging easier
comments: true 
categories: programming
tags:
 - android
 - adb
--- 

[ADB][1] is a command-line tool that lets you communicate with a device. It is a client-server program that includes three components:
* A client, which sends commands. The client runs on your development machine. You can invoke a client from a command-line terminal by issuing an adb command.
* A daemon (adbd), which runs commands on a device. The daemon runs as a background process on each device.
* A server, which manages communication between the client and the daemon. The server runs as a background process on your development machine.

One of the problems that sometimes arises with using ADB is to connect to the device without it being connected to your local machine via a USB cable. However, ADB makes it possible to connect the device 
over WiFi because of it's simple server-client architecture. 


To connect to ADB over WiFi, you just need to follow these steps.

Pre-requisites
* Connect the device to your development machine via a USB cable.
* Verify the device can be used by running `adb devices`. The device should be listed.

Next, run the following commands:

* You will need the IP address of the device you need to connect to. To get the IP address, run:
{% highlight bash %}
adb shell ip addr show wlan0 | grep "inet\s" | awk '{print $2}'
{% endhighlight %}

* To start ADB over TCP/IP connection, run:
{% highlight bash %}
adb tcpip 5555
{% endhighlight %}

* Now, just start the ADB connection by running: 
{% highlight bash %}
adb connect 192.168.0.25:5555
{% endhighlight %}

Don't forget to replace the above IP and port with the IP of your device and the port you actually used in the commands above. 

That's it. You should now be able to remove the cable connecting the device and see that adb commands will continue to function. 

 [1]: https://developer.android.com/studio/command-line/adb
