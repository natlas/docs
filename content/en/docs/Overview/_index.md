---
title: "Overview"
linkTitle: "Overview"
weight: 1
description: >
  What Natlas is, and what it isn't
---


## What it is

Natlas is an open source continuous reconnaissance platform which prioritizes identifying network exposure, rather than identifying vulnerabilities. While the processes may be similar, Natlas emphasizes identifying network exposure and letting the user find insights from that. Natlas is a host-oriented scanning platform, which means that jobs are designated by ip address. This means that one scan provides a comprehensive view of a host at a given point in time. Contrast this with port-oriented scanning, in which a job is designated by one or more ports and then scans a large range of addresses.

Natlas performs port scans using the popular [Nmap] scanner and then stores that data in a structured format that allows for easy search. Using Nmap also allows for extensibility using the powerful Nmap Scripting Engine, instead of requiring custom tooling to be written into the Natlas agent itself. As if port scanning wasn't enough, Natlas also takes screenshots of web services using [Aquatone] and vnc services using [Vncsnapshot], providing the option to visually browse your exposure right along side your port scan data.

## What it isn't

Natlas is **not** a vulnerability scanning platform. In fact, at this time, there is no consideration of CVEs anywhere in the project. While Natlas will almost undoubtedly help discover security flaws and policy violations, it will not spit out a list of machines ranked by how vulnerable they are.

## Why do I want it

* **What is it good for?**:
  * Monitoring internal asset exposure via internal network scanning
  * Monitoring external asset exposure via known IP address ranges
  * Tracking changes in exposure over time

* **What is it not good for?**:
  * Natlas does not do port-oriented scanning, which means it isn't a replacement for popular internet scanning tools such as masscan, zmap, etc.
  * Natlas does not do point-in-time scans. If you want that, you're probably better suited using one of the tools from the previous point.
  * Natlas handles work distribution, but it does not do compute instance orchestration.

* **What is it *not yet* good for?**:
  * Natlas currently does not do DNS based asset discovery
  * Natlas currently has scaling limitations that start to choke the server around 2400-2500 scanning threads
  * Natlas currently does not produce automatic statistics about the scan data
  * Natlas currently does not provide any alerting mechanism to get notified when new scans match an alert rule

## Where should I go next

* [Play with the demo]: Check out our demo site and see if Natlas is right for you
* [Getting Started]: Get started with Natlas
* [Tutorials]: Check out some tutorials that walk step by step through setting up Natlas

[Nmap]: https://nmap.org
[Aquatone]: https://github.com/michenriksen/aquatone
[Vncsnapshot]: http://vncsnapshot.sourceforge.net/
[Play with the demo]: https://natlas.io
[Getting Started]: /getting-started/
[Tutorials]: /tutorials/
