---
title: "Searching Natlas"
date: 2020-08-16
weight: 2
description: >
  Learn how to search Natlas data like a pro
---

Natlas search is powered by [Elasticsearch](https://www.elastic.co/), and the syntax that the search box uses is currently equivalent to an Elastic [Query string query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html). For a deeper understanding of the query string syntax, please consult that page. A key thing to remember when building queries is that, by default, more search terms results in a narrower result set. Each additional search term serves to further reduce the result set.

The Natlas elastic mapping currently makes a lot of things available to you to search, but it may be less than intuitive or unnecessarily verbose because there are no abstractions or shortcuts for common queries. So this page should help you to build more effective queries.

If you choose not to specify any fields to query for, it will default to searching the `nmap_data` field, which is roughly equivalent to grepping through your `scan.nmap` files.

## Basic Syntax

This section will not go into depth about the query string syntax, but will cover some of the most common operators.

Natlas supports `AND`, `OR`, and `NOT` for combining queries. The default operator for Natlas is `AND` when multiple search terms are present, (separated by a space). This is important to remember, as the default for Elastic is typically `OR`. If you'd like to use `OR`, you can do so like so: `ports.port:80 OR ports.port:443`, which returns hosts that have port 80 or port 443 open.

`NOT` can also be shortened to `!`, e.g. `!tags:"Corpnet-A"` to return only results that are not tagged with "Corpnet-A".

Natlas also supports wildcard operators when searching text fields, such as `hostname:*.example.com` to match all subdomains of example.com, or `hostname:example.???` to find all example.tld domains that have a 3 letter tld.

## Metadata

Metadata terms are those that are related to the Natlas scan but are not directly related to the acquired scan data.

### Scan Information

#### scan_reason

**Example:** `scan_reason:manual`

**Description:** scan_reason is used to differentiate between automatic scanning, manually submitted scans, and user-requested rescans. The valid options for this are `manual`, `automatic`, and `requested`.

#### tags

 **Example:** `tags:Digital Ocean`

 **Description:** tags are applied to scope ranges to automatically group different groups of IP addresses together. These are completely configurable per deployment.

#### scan_id

 **Example:** `scan_id:b8befe174067010b6182c3ca28baf095`

 **Description:** scan_id is an automatically generated token that uniquely identifies a scan. This is particularly useful if you want to share specific scans with someone.

#### ip

 **Example:** `ip:10.0.0.0/8`

 **Description:** ip is the IP address targeted by the scan, and supports CIDR notation so that you can search for entire CIDR ranges at a time. The example shows all scans from the `10.0.0.0/8` network.

#### hostname

 **Example:** `hostname:*.example.com`

 **Description:** hostname is determined by nmap as the rDNS (PTR record) value for a given IP address. The example query shows all scans that have a PTR record for a subdomain of example.com

### Automatic Queries

These terms are inherent to all search results by default and do not need to be explicitly defined:

#### port_count

 **Example:** `port_count:>0`

 **Description:** port_count is calculated based on the number of open ports that a scan reports. The example query shows all scans that have more than 0 ports open.

#### is_up

 **Example:** `is_up:True`

 **Description:** is_up is reported by nmap for whether or not it determined the host was up. This defaults to True for search queries, because looking at hosts that we don't have data for isn't very interesting.

### Timing

These search operators all pertain to date related fields that are inherent to the Natlas scan documents. These all support operators like `>`, `<`, `<=` and `>=` to do simple comparisons comparisons.

#### ctime

 **Example:** `ctime:>=2020-01-01`

 **Description:** ctime is the creation time of the scan document, which means when the server inserts it into Elastic. The example query shows all scans that were inserted on or after January 1st, 2020.

#### scan_start

 **Example:** `scan_start:2020-01-05`

 **Description:** scan_start is provided by the agent and tells us when the agent starts doing its work. The example query shows all scans that started (but not necessarily finished) on January 5th, 2020.

#### scan_stop

 **Example:** `scan_stop:2020-01-05`

 **Description:** scan_stop is provided by the agent and tells us when the agent stopped doing it's work. The example query shows all scans that stopped (but not necessarily started) on January 5th, 2020.

#### elapsed

 **Example:** `elapsed:>60`

 **Description:** elapsed is calculated based on scan_start and scan_stop and tells us how long the agent spent working on a particular job. The example query shows all scans that took longer than 60 seconds to perform.

### Agent

These search terms pertain to the agent that performed the work.

#### agent

 **Example:** `agent:44040760a19f5dc6`

 **Description:** agent is the identification token of the agent that performed the work. If no token is present, this value will be `anonymous`.

#### agent_version

 **Example:** `agent_version:0.6.11`

 **Description:** agent_version is the configured `NATLAS_VERSION` of the agent that reported the work. In `0.6.x` this is used to determine how to render the scan data.

## Screenshots

These search terms all focus on finding screenshots that meet specific criteria. You'll notice that all of these are prepended with `screenshots.`, which is how you specify the specific field in the document that you want to scan when it isn't at the top level of the document. This will be a recurring theme for the rest of this documentation.

### screenshots.host

 **Example:** `screenshots.host:*.example.com`

 **Description:** Returns scan results that have a screenshot for the provided host. The host may be a DNS name or an IP address. The example query returns scan results for all subdomains of example.com (as determined by their PTR record) that have a screenshot.

### screenshots.port

 **Example:** `screenshots.port:8089`

 **Description:** Returns scan results that have a screenshot for the specified port. The example query returns all scans that have a screenshot for port 8089.

### screenshots.service

 **Example:** `screenshots.service:VNC`

 **Description:** Returns scan results that have a screenshot for the specified service. Note that these values are case sensitive and must be uppercase at this time. The currently supported services are `HTTP`, `HTTPS`, and `VNC`.

### screenshots.hash

 **Example:** `screenshots.hash:<sha256>`

 **Description:** Returns scan results that have a screenshot with the specified hash. This is helpful for finding all hosts that have an identical service running. Note that this is an exact match, not a fuzzy hash.

### screenshots.thumb_hash

 **Example:** `screenshots.thumb_hash:<sha256>`

 **Description:** Returns scan results that have a screenshot whose thumbnail matches the specified hash. This is mostly equivalent to `screenshots.hash` except it searches thumbnails instead of full size images.

## Ports

Searching port data is probably the most important thing you'll want to do with Natlas, but it can also be the most complex. Values in the ports structure are sometimes deeply nested and complicated to build queries for.

### ports.id

 **Example:** `ports.id:tcp.80`

 **Description:** This is the port id from nmap, which is a combination of the protocol and the port number. The example query searches for ports that have tcp port 80 open.

### ports.port

 **Example:** `ports.port:80`

 **Description:** This is the port number from nmap, without regard for the protocol. The example query searches for all hosts that have port 80 open, regardless of tcp or udp.

### ports.protocol

 **Example:** `ports.protocol:tcp`

 **Description:** This is the port protocol from nmap. The example query searches for all hosts that have a tcp port open, regardless of what number it is. Largely, this query is probably not going to be very helpful since all results will probably have a tcp port open.

### ports.state

 **Example:** `ports.state:open`

 **Description:** This is the port state from nmap. The example query searches for all hosts that have a port open. Again, probably not especially helpful.

### ports.reason

 **Example:** `ports.reason:syn-ack`

 **Description:** This is the reason the port is included by nmap. `syn-ack` is probably going to be the most common, and therefore the least useful.

### ports.banner

 **Example:** `ports.banner:Apache`

 **Description:** This searches the banner string produced by nmap. The example shows all hosts with an Apache service reported in the banner grab.

### Service

Remember in the last section how I said that port data is complex and sometimes deeply nested, well, welcome to the first next layer. This section pertains to information about the service running on a port. Basically what this does is provides a more nuanced approach to searching the results of nmap's service fingerprinting, rather than searching the text field `ports.banner` from the previous section.

#### ports.service.name

 **Example:** `ports.service.name:http`

 **Description:** This is the service name that nmap determines from the port. With service fingerprinting enabled, this may originate from the fingerprints. Otherwise it will fall back to what's in the provided services database. The example will find hosts that have at least one http service running.

#### ports.service.product

 **Example:** `ports.service.product:Minecraft`

 **Description:** This is the product that nmap determines from the service fingerprint. The example query will find instances of Minecraft running (notice that this is case sensitive).

#### ports.service.version

 **Example:** `ports.service.version:1.16.1`

 **Description:** This is the product version that nmap determines from the service fingerprint. The example query will find anything with a version string of `1.16.1`

#### ports.service.ostype

 **Example:** `ports.service.ostype:Linux`

 **Description:** This is the OS type determined by nmap. The example query will find anything that is determined to be Linux.

#### ports.service.conf

 **Example:** `ports.service.conf:>9`

 **Description:** This is the confidence level of nmap that the service it identified is actually that service. This is an integer value between 0 and 10, with 10 being the most confident about the accuracy of the reported service.

#### ports.service.cpelist

 **Example:** `ports.service.cpelist:"cpe:/a:apache:http_server:2.4.29"`

 **Description:** This is the [CPE](https://nvd.nist.gov/products/cpe) identifier for the identified service. The example shows us Apache http servers running version 2.4.29.

#### ports.service.method

 **Example:** `ports.service.method:probed`

 **Description:** This is the method that nmap used to determine what the service was. `probed` means that it was the result of the probes, `table` means that it was grabbed from the services database.

#### ports.service.extrainfo

 **Example:** `ports.service.extrainfo:"ubuntu"`

 **Description:** Extraneous information retrieved by the nmap service fingerprint that doesn't fit directly into one of the other categories. The example looks for instances of ubuntu, indicating that it's likely to be an Ubuntu linux server.

#### ports.service.tunnel

 **Example:** `ports.service.tunnel:ssl`

 **Description:** Whether or not the host has a port that uses a tunnel, such as ssl.

### Scripts

Everything in this section involves searching the output of the nmap scripting engine. Most script outputs are stored as text blobs that you'll have to search through.

#### ports.scripts.id

 **Example:** `ports.scripts.id:http-git`

 **Description:** Look for hosts that have ports open where a specific script was executed. The example will look for hosts that have results for the `http-git` script (looking for `/.git` folders in http web roots)

#### ports.scripts.output

 **Example:** `ports.scripts.output:"github.com"`

 **Description:** Perform string searches on the output of scripts. The example will look for script output where "github.com" appears, such as in the previous `http-git` example.

### SSL

SSL Data is collected by the nmap scripting engine and is also available with the `ports.scripts.id:ssl-cert` query, however these certificates are important enough, and complicated enough, to be parsed out into structured fields. But alas, SSL data is the most deeply nested of all of the search terms, so prepare yourselves.

#### ports.ssl.md5

 **Example:** `ports.ssl.md5:da979cf70ae708ffb1d8a247a0cf5acb`

 **Description:** Looks for hosts that have an ssl certificate with a specific md5 hash.

#### ports.ssl.sha1

 **Example:** `ports.ssl.sha1:28cfccaf54b309b6d7c58b400ce8262f3121a495`

 **Description:** Looks for hosts that have an ssl certificate with a specific sha1 hash.

#### ports.ssl.notAfter

 **Example:** `ports.ssl.notAfter:<2020-08-16`

 **Description:** Look for hosts that have ssl certificates with a particular expiration date. The example looks for certificates with an expiration date prior to August 16th, 2020.

#### ports.ssl.notBefore

 **Example:** `ports.ssl.notBefore:>2020-01-01`

 **Description:** Look for hosts that have ssl certificates with a particular notBefore validity date. The example looks for certificates that were issued after January 1st, 2020.

#### ports.ssl.sig_alg

 **Example:** `ports.ssl.sig_alg:sha256WithRSAEncryption`

 **Description:** Look for hosts that use a specific signature algorithm for the ssl certificate. The example looks for signature algorithm `sha256WithRSAEncryption`.

#### ports.ssl.pubkey.bits

 **Example:** `ports.ssl.pubkey.bits:<2048`

 **Description:** Look for hosts that have a pubkey with a particular size. The example looks for hosts that have pubkeys that are less than a 2048-bit key.

#### ports.ssl.pubkey.type

 **Example:** `ports.ssl.pubkey.type:ec`

 **Description:** Look for hosts that use an ssl pubkey of a specific type. The example looks for ssl pubkeys using Elliptic Curve (ec).

#### ports.ssl.issuer.commonName

 **Example:** `ports.ssl.issuer.commonName:"Let's Encrypt Authority X3"`

 **Description:** Look for hosts that have a certificate issued by a specific issuer. The example looks for certificates issued by Let's Encrypt's Authority X3 issuer.

#### ports.ssl.issuer.countryName

 **Example:** `ports.ssl.issuer.countryName:us`

 **Description:** Look for hosts that have a certificate issued by an issuer in a specific country. The example looks for certificates issued by an issuer in the United States.

#### ports.ssl.issuer.organizationName

 **Example:** `ports.ssl.issuer.organizationName:"Let's Encrypt"`

 **Description:** Look for hosts that have a certificate issued by a specific organization. The example looks for certificates issued by Let's Encrypt.

#### ports.ssl.subject.commonName

 **Example:** `ports.ssl.subject.commonName:*.example.com`

 **Description:** Look for hosts that have a certificate issued to a specific subject. The example looks for certificates issued to subdomains of example.com.

#### ports.ssl.subject.altNames

 **Example:** `ports.ssl.subject.altNames:*.example.com`

 **Description:** Look for hosts that have a certificate with Subject Alternate Names for a specific subject. The example looks for certificates issued that are also valid for subdomains of example.com.
