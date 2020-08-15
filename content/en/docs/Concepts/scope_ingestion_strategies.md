---
title: "Scope Ingestion Strategies"
linkTitle: "Scope Ingestion Strategies"
weight: 4
description: >
  How does Natlas know what's in scope?
---

## Current Strategy

Currently, Natlas only supports manually imported scope in the form of CIDR ranges. This may work well for scans of RFC1918 address space, as well as for larger organizations that have static CIDR ranges on the internet. But it really doesn't work well for smaller organizations using cobbled together address space, let alone cloud-native companies.

## Possible Strategies

* (Priority) DNS enumeration ingestion
  * Given one or more apex domains, perform subdomain enumeration and use valid subdomains to populate scope.
  * Requires the ability to:
    1. Automatically add ip addresses of new subdomains to scanning
    1. Add new subdomains to a review queue where an administrator decides to scan or not
    1. Blacklist DNS names such that no matter what ip they point to, we do not scan them
  * Can be automated fairly well with the use of amass for enumeration and background workers that regularly re-resolve already known records (and keep a history of these re-resolves)
* Authenticated DNS zone transfer ingestion
  * If you are an administrator or on the IT team for an organization, you may wish to perform authenticated zone transfers in order to maintain an authoritative source of DNS records to scan for
  * Likely a relatively low priority option, and could be easily automated with a Natlas API and a curl script in the meantime
* Cloud Provider Events API(s)
  * Receive / subscribe to events for your cloud provider in order to receive notification of new hosts being deployed and then automatically feed these into natlas scope
  * Works really well for scanning primarily cloud environments
  * Likely requires plugins for GCP, AWS, Azure, etc
* Cloud Provider Enumeration API(s)
  * Periodically (on a configurable time) run tooling that queries cloud provider APIs to build a list of hosts to be ingested for scanning
  * Can create a minor gap in receiving information about new hosts when compared to the previous events api source
  * Similarly will require plugins for GCP, AWS, Azure, etc

## Scope Ingestion Methods

* Add or remove a single item from scope/blacklist
* Add or remove a batch of items from scope/blacklist
* Replace the entire scope/blacklist for a given ingestion source
  * We don't want DNS zone transfer updates to be additive, for instance. We know these will be the hosts to scan and will likely want to completely invalidate the previous scope, opposed to diffing the previous scope and the new scope and then removing/adding as necessary. Functionally they would result in the same thing, and in probably very comparable performance.
  * Note the addition of "ingestion source", which indicates that if we allow automated ingestion, we will need to include an ingestion source so that things don't necessarily overwrite one another.
  * This is basically just "Add or remove a batch of items" run back to back, first removing and then adding, based on a query for the ingestion source.
* Remove the entire scope/blacklist for a given ingestion source
  * Similar to replace, create a query for scope from a given ingestion source and then batch remove those items.

## Caveats

Automating scope ingestion too frequently will wreak havoc on the PRNG, preventing us from reliably reaching full scope coverage, which was the whole point of the PRNG in the first place. This means we need to discuss mechanisms for new scope to be added without interrupting the existing PRNG cycle, but without necessarily waiting until it's completely finished before we start scanning the new scope. One option for this is that changes to the scope go into a second scan manager, or a scan manager cache of sorts, and then jobs are distributed from between them. The problem is that this may grow out of hand if scope ingestion is happening too frequently, unless the scan manager says "Okay those are in my current scope so I'm not going to cache them, but these other ones are new so we'll add them to our secondary job queue". Anyways, this is just something to think about as we move towards implementing this.
