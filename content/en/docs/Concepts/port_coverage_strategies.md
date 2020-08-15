---
title: "Port Coverage Strategies"
linkTitle: "Port Coverage Strategies"
weight: 4
description: >
  How does Natlas determine what ports to scan?
---

## Current Strategy

Currently, we have only one port coverage strategy - Define ports to be given to the natlas-agent via the natlas-services file. This feeds directly to nmap and we scan all the ports in the natlas-services file. This has the advantage of being simple while also allowing per-deployment customization of what ports to scan. But it has the disadvantage of not scaling up very well; you could add another 5000 ports but nmap is not a very fast scanner by default. So basically you don't identify anything _new_ with this approach, only the things that you think you might find ahead of time.

## Proposals

Ideally we'd like to have all of these capabilities to allow for flexibility in configuration, as each has it's own drawbacks and advantages.

### The Multi Armed Bandit

It would be possible to design scanning behavior that models the scarceness of scanning resources and optimizes its use, given a given scanning scope. Since every scan costs CPU and networking resources as well as latency, and any scan on any port has some statistical chance at reward that depends on the port, Natlas could model the scanning problem as an instance of the well studied [Multi-Armed Bandit problem](https://en.wikipedia.org/wiki/Multi-armed_bandit) for which optimal and rapidly converging strategies are known.

Effectively, when it is time to scan a particular host - or set of hosts - the Natlas server would utilize information it has previously collected to fine tune its scanning ranges. In a "vanilla" multi-armed bandit model, the decision of the server to scan hosts on specific ports would only include information about which ports have been seen before as statistical aggregates across the entire scope. However, if a variations of the problem - such as "contextual bandits" - were used as the model, it would be possible to combine scope tags, ASNs, and/or other information with port statistics to narrow in on favorable port scan targets.

The resulting experience would be that a Natlas administrator would set a certain number of ports each agent would be expected to scan, but not specify which ports these should be or could optionally provide "seed ports" that are initial exposure guesses to start the scan off with. Natlas would then - over the course of several scanning Epochs - use scan results it is collecting as feedback to refine what ports it is asking the agents to scan. Effectively, over time, this would result in scanning the most common ports in _that specific_ scope, without needing to scan all 65k ports on all hosts all the time.

The algorithms that solve this problem always allocate a few resources for "exporation", which means that even after a large number of scanning epochs has passed, new scans will balance the risk of scanning ports not likely to result in discoveries against the need to determine probe unlikely ports to learn whether historically collected statistics are still accurate.

**Pros:**

* Results in an efficient use of scans
* Works in scenarios where few common ports are active, and they may not be known in advance
* Can be tuned to be more aggressive or conservative and to learn faster or slower
* Can start out identical to default port lists (e.g. nmap)
* Can be combined with other approaches so that there are a "must have" port scan, and remaining ports are "learned"

**Cons:**

* Can not guarantee that specific hosts/port combinations will be scanned
* Host history will not be garunteed to be an apples-to-apples comparison
* This introduces state into the server portion of Natlas
* The architecture would need to be redesigned to support dynamic per-scan port and service targets
* Bugs in scanning logic or deployment will result in artifacts that get "learned". For example, if the scanner loses all packets inbound from the target scope, the strategy will learn that all ports are equally as good (since scan results indicate that all ports are down for all hosts)

### Pre-flight Masscan

This is a simple proposal for an agent to run masscan for all ports against each host before passing the open ports to nmap for deeper scanning. At a rate limit of 10000pps, this would add approximately 65 seconds per agent job. It would take some time out of the nmap scanning phase, because nmap would receive a list of ports that should be open, so it shouldn't be spending much time scanning ports that aren't open. It would also have the benefit of not relying on nmap's host discovery, which can often be tricked into reporting a host as down. The option currently exists to tell nmap to skip it's host discovery phase (`-Pn`), but this is also slow because it means it checks every port no matter what. It is quite common across recon enthusiasts to use masscan prior to nmap, though the typical model for masscan is a port-oriented strategy which scans large ranges of network space for a limited number of ports, rather than what we're doing here.
