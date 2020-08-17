---
title: "Managing Scope"
linkTitle: "Managing Scope"
date: 2020-08-16
weight: 2
description: >
  Learn how to manage your scanning scope
---

## Importing Scope

Before your Natlas deployment can do anything useful, you'll need to import some scope addresses for your agents to scan. Natlas provides two paths for doing this: CLI and Web. The CLI demos in this documentation assumes that your natlas server container is named `natlas_server`.

### The Scope File

Your scope file will be a csv file with each row representing one CIDR network, optionally followed by a list of comma-separated tags to describe that CIDR network. E.g.

```text
192.168.0.0/16,home,class-b
172.16.0.0/16,vpn,class-b
10.0.0.0/8,overlay,class-a,super cool network
192.168.0.100/32,pihole
192.168.100.1/32,modem
192.168.0.1/32,router
```

An observant reader will notice that the last 3 addresses all fall within the range of the first network. This is fine, more specific addresses will be tracked individually so that you can tag them with whatever you want. Just note that if you later choose to exclude the `192.168.0.0/16` network, all addresses that fall inside that range will also be excluded, regardless if they are also individual items in the rest of the scope.

### Importing by CLI

Importing via CLI is pretty straight forward but requires putting your scope file into the persistent mounted directory. Since the server requires this directory to store media files already, it shouldn't be a big deal to drop your bootstrap scope into that directory. The data will be available in `/data` inside the container.

We put our scope file into our persistent storage directory and then we can import it with flask like so:

```bash
docker exec -it natlas_server flask scope import --verbose /data/myscopefile.txt > /data/import_results.json
```

Remember, the `/data/myscopefile.txt` and `/data/import_results.json` are relative to the docker container, not to the host filesystem.

### Importing by Web

Maybe manipulating the flask application through the docker layer is too verbose and you don't want to do it. That's fine and dandy, Natlas provides a web interface for admin users to import scope items individually or in bulk at the `/admin/scope` url. The bulk import works the same way as the file import on CLI, except you paste the contents of the file rather than uploading the file itself.

![Scope Management Panel](https://i.imgur.com/KWByFj5.png)

![Scope Import](https://i.imgur.com/ODGWNVB.png)

## Exporting Scope

### Exporting by CLI

The CLI claims to support exporting scope, however there is currently a bug ([#246](https://github.com/natlas/natlas/issues/426)) that prevents this from working correctly when your scope items are tagged.

### Exporting by Web

As you can see above in the scope management panel, there's a simple export button. You can also reach this directly by visiting `/admin/export/scope` while logged in as an Admin. This produces a simple line-separated list of in-scope networks.

## Disabling Scope

Disabling scope is a way to remove specific addresses or subnets from being scanned. This can be important if you're scanning things that are likely to tip over when being touched by nmap, or things that are deemed "too risky to scan." You can easily add new network blocks to the exclusion list, or simply move existing network blocks.

![Disabling existing scope](https://i.imgur.com/o7NGFHa.png)
