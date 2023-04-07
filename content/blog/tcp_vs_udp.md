---
title: "TCP vs UDP"
date: 2021-10-22T14:36:35-06:00
draft: false
---


Today at work I was challenged to find out why our monitoring software was constantly receiving errors and false alerts from our application. While I was diving into the logs I came across:

> snmpd[1311]: Connection from UDP

This led to two questions:
* Why are there so many connections using UDP?
* Why are these connections being made with the same timestamp as the false alerts?

Since I work with some pretty incredible individuals I was given a crash course followed by several suggestions on where to further my learning. **First what are TCP and UDP and what do they do?** Both of these are protocols that send bits of data over the internet. They both build on top of the internet, which means they are both sending packets (bits) to an ip address. **TCP is Transmission Control Protocol** TCP is a transmission that requires an acknowledgement. This is the most common transmission over the internet. It is used when you load a web page, because the browser sends TCP packets to the IP address asking them to send the web page. The web server responds by sending packets which your browser forms into the web page you are viewing. When there is a stream of packets being sent TCP will number them so if the acknowledgement with the correct response isn’t given the stream will resend.

**UDP is User Datagram Protocol**UDP is a transmission that does not require an acknowledgement. Which means that it is less reliable, but faster. There is no guarantee that you receive all the packets and if you don’t that is too bad. There is no error checking and there is no guarantee you receive the packets in the order they were sent out. You are however, guaranteed time. The error checking and constant acknowledgement takes time.

**Examples of when to use one or the other**When downloading a file from a website the packets need to be an a specific order. if there is any latency in your internet then you are likely not going to receive these packets in the correct order and likely not all of them the first time they are sent. This requires TCP to be used. However, if you are watching the Thursday Night football as I am right now. Then you want a constant stream of packets rather than every one of them being acknowledged. If you miss a packet then the football may appear to teleport forward rather than a fluid movement. This will allow for almost live feed rather than a playback after everyone of them have been sent over and verified.

Tomorrow I will be working with IT to see if the packets sent during the UDP is the error messages and how to better configure our monitoring to be as optimized as possible.

For more information click [here](http://www.howtogeek.com/190014/htg-explains-what-is-the-difference-between-tcp-and-udp/)