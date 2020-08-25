---
title: "Community Queries"
linkTitle: "Community Queries"
date: 2020-08-16
weight: 2
description: >
  Take your Natlas queries to the next level
---

This page serves as a community collection of Natlas queries that are good at finding specific items of interest. Some may be fairly common queries that produce interesting results, while others may be significantly more advanced and narrow in on a very specific thing. Share your dorks with us here by simply clicking "Edit this page" on the right hand side of the page.

## From the Staff

### Expired SSL Certificates

**Query:** `ports.ssl.notAfter:now`

**Description:** This query makes use of Elastic's [range on date](https://www.elastic.co/guide/en/elasticsearch/reference/6.6/query-dsl-range-query.html#ranges-on-dates) query capability, where now is automatically expanded out to the current date and time.

### SSH Password Auth

**Query:** `ports.scripts.id:ssh-auth-methods password`

**Description:** This query looks for hosts where the [ssh-auth-methods](https://nmap.org/nsedoc/scripts/ssh-auth-methods.html) nmap script has run and then filters it down further to only hosts who also contain "password" in their output. This does have the potential for false positive, as queries apply to hosts rather than to individual ports (so if password appears in the response of another port, false positive).

### Exposed Docker Daemons

**Query:** `ports.port:2375 OR ports.port:2376`

**Description:** This query looks for open ports that are commonly used by the [Docker daemon](https://docs.docker.com/engine/reference/commandline/dockerd/). An exposed Docker daemon can result in [security incidents](https://unit42.paloaltonetworks.com/attackers-tactics-and-techniques-in-unsecured-docker-daemons-revealed/) if care is not taken.

### Exposed Git Repositories

**Query:** `ports.scripts.id:http-git`

**Description:** When a `.git` directory can be found over http, it's usually worth exploring to see if the developers left credentials lying around. Read more about the [http-git](https://nmap.org/nsedoc/scripts/http-git.html) script.

### Anonymous FTP Servers

**Query:** `ports.scripts.id:ftp-anon`

**Description:** Exposed FTP servers can be pretty mundane, since many sites will intentionally provide some file downloads over FTP. You may wish to append interesting directories to this search, such as `ports.scripts.id:ftp-anon etc`. Alternatively, you can further filter with `NSE: writeable`, per the [ftp-anon](https://nmap.org/nsedoc/scripts/ftp-anon.html) documentation, e.g. `ports.scripts.id:ftp-anon "NSE: writeable"`

### Exposed MongoDB

**Query:** `ports.scripts.id:mongodb-databases !"codeName = Unauthorized" !"ERROR: Script execution"`

**Description:** This query is slightly more complex than the others so far. First we look for hosts where the [mongodb-databases](https://nmap.org/nsedoc/scripts/mongodb-databases.html) script ran, then we filter out results where we're unauthorized, and finally we filter out results where script execution failed. When we put it all together, there's a high chance that the results will have exposed unauthenticated databases.

### Exposed Elasticsearch

**Query:** `"You Know, for Search"`

**Description:** Look for exposed Elasticsearch nodes by searching for their tagline, which will show up in the `fingerprint-strings` wherever an elasticsearch node can be reached. You definitely don't want these to be available on your network, lest you get [Meowed](https://www.bleepingcomputer.com/news/security/new-meow-attack-has-deleted-almost-4-000-unsecured-databases/)


## From the Community

Oops! Looks like we haven't received any community contributions yet!
