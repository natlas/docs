---
title: "Concepts"
linkTitle: "Concepts"
weight: 4
description: >
  Waxing conceptual about Natlas
---

Natlas has a number of conceptual things to consider, including how we reach 100% host coverage for a given address space, as well as how we can improve upon our port coverage for those hosts. There are also some terms that should be clarified here:

* **Scan Cycle**: A scan cycle is one complete iteration of scanning the entire scope.
* **Scan Group**: A scan group is a collection of scope items that are grouped together for the purposes of scanning. This has barely been implemented so far, but will be relevant for multi-tenant deployments or deployments where agents are scanning disparate networks that cannot talk to one another and need a way to identify which set of addresses to scan.
* **Scope**: Scope is a CIDR range (anything from /0 to /32 (or /128 for IPv6)) that is included for scanning.
