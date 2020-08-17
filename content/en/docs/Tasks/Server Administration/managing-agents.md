---
title: "Managing Agents"
linkTitle: "Managing Agents"
date: 2020-08-16
weight: 3
description: >
  Learn about the tools available to manage the way the agents work
---

## Configuring Port Coverage

Port coverage is configured via a server side [nmap-services](https://nmap.org/book/nmap-services.html) file, which is used by nmap to know what ports to scan. Natlas maintains it's [own version of this file](https://github.com/natlas/natlas/blob/main/natlas-server/defaults/natlas-services), which is shipped with the application. We use it to add new common or interesting ports that the default nmap-services doesn't cover. You're welcome to add or remove as many ports as you want from this file.

To access this on your server, visit the `/admin/services` endpoint while logged in with an admin account. It'll look something like this:

![Natlas Services Page](https://i.imgur.com/I3UdVfJ.png)

From here, you can add individual new services via the form, or you can upload a new services file to completely replace the current one.

## Configuring Scans

Everything about scans, except what ports are covered, can be found in the `/admin/agents` url. This includes what nmap settings to use as well as whether or not you want them to collect screenshots of web services or vnc services.

![Agent configuration page](https://i.imgur.com/9G7UeZp.png)

### Supported Nmap Options

Natlas doesn't support raw nmap command editing, because it makes a goal to not allow arbitrary execution on the agents. As such, we currently only support a limited subset of nmap options. They are as follows:

* [Version Detection](https://nmap.org/book/man-version-detection.html) (-sV)
* [OS Detection](https://nmap.org/book/man-os-detection.html) (-O)
* [Limit OS Scan](https://nmap.org/book/man-os-detection.html) (--osscan-limit)
* [Enable Scripting Engine](https://nmap.org/book/nse-usage.html) (--script)
* [Report Open Ports Only](https://nmap.org/book/man-output.html) (--open)
* [Bypass Host Discovery](https://nmap.org/book/man-host-discovery.html) (-Pn)
* [Scan TCP+UDP](https://nmap.org/book/scan-methods-udp-scan.html) (-sUS) **Note:** This requires your services file to have udp ports in it.

### Nmap Scripts

The list of scripts to scan is stored separately so that you can easily add and remove scripts. This field allows individual script names as well as script categories. Check out the [nmap documentation](https://nmap.org/book/man-nse.html) for more information about this, or checkout the [NSE Library](https://nmap.org/nsedoc/lib/nmap.html) for the list of scripts that are shipped with nmap.

At this time, Natlas does not support sending custom scripts to the agents due to the security design of the agents. Custom NSE scripts effectively allows arbitrary lua execution on the agent, and we're not trying to build a botnet. If you own all of the agents in your natlas deployment, you could manually distribute your custom scripts to the agents and then enable it in the server.

**Important:** If you specify a script that doesn't exist, all of your scan data will start to automatically fail. The agent currently doesn't maintain a list of scripts it knows about, nor does it parse the error output when nmap fails to initialize.

### Screenshots

Natlas currently supports screenshotting two service types: HTTP and VNC.

#### HTTP Screenshots

Natlas uses aquatone for web screenshots, which automatically reads the output of the nmap file using aquatone's `-nmap` flag. This means that it will attempt to screenshot hostnames if available, and IP addresses otherwise. Natlas currently does not support any custom flags for aquatone. It only supports a process timeout value, which can be used to keep your agents from lingering too long on any given task. If the timeout value is reached, the process will be killed and the output will not be processed.

#### VNC Screenshots

Natlas currently uses vncsnapshot for vnc screenshots, and only on port 5900. A future version will support VNC on arbitrary ports and will hopefully provide more control over the screenshots you get to take. Like HTTP Screenshots, VNC screenshots also only support a process timeout value, after which the process will be killed and the output abandoned.
