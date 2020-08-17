---
title: "Security Considerations"
linkTitle: "Security Considerations"
weight: 5
description: >
  What security considerations does Natlas keep in mind?
---

A security tool, if you can call Natlas that, wouldn't be very good if it introduces more vulnerabilities than it identifies. As such, we keep security in mind through every step of development. We'll talk about the way we use containers, the boundaries between the server and the agent, and how to develop new features.

## Secure By Default

As can be seen from many other open source projects, if a project is provided that doesn't enable security out of the box, then there's a very good chance that security will never get enabled. Natlas is building a map of all the interesting things in your network, so it makes sense that you might want to ensure that only authorized people can look at it. That's why Natlas ships with both user and agent authentication required, as well as separation between admin users and regular users. We believe that security shouldn't be an expansion to the project, nor should it be opt-in.

## Natlas Containers

Natlas uses containers for both the agent and the server. Containers by themselves shouldn't be inherently considered a security boundary, but we can use them to at least ensure that we aren't running with more privileges than we need.

### Server Container

The server container does launch as root, technically. This is so that it can create directories in the persistent storage that it requires and make sure that it can write to those files. Through normal operation, the entrypoint of the container will make sure the files it needs exists, make sure the database is up to date, and then run the server as the `www-data` user. This entrypoint script may be removed in the future and it'll be the user's responsibility to ensure that the app can write to the persistent storage mount.

We also make an effort to provide slim containers, however the application requires the python interpreter to run, so if it were compromised then the python interpreter would be available to do anything an attacker might want.

### Agent Container

The agent container launches exclusively as the `www-data` user. The agent uses nmap and aquatone, we have to provide some specific flags in order to function correctly. `Nmap` runs with the `--privileged` flag, which lets us do SYN scans without needing to be root, but requires `CAP_NET_ADMIN` when the container is launched (`--cap-add=NET_ADMIN`). Since aquatone relies on chromium, we have to consider how to launch chromium from within docker. Many searches on this topic will tell users "oh just run with --no-sandbox" but we'd like to be a little less reckless than that. Thankfully, [Jess Frazelle](https://github.com/jessfraz) has gone to a lot of effort to dockerize all sorts of applications, including Chrome. Natlas makes use of Jess' [seccomp profile](https://github.com/jessfraz/dotfiles/blob/master/etc/docker/seccomp/chrome.json) for Chrome, which allows us to run chrome in our container as expected.

## Server-Agent Boundary

One might look at a distribution platform like this and think "oh yeah, if we just compromise the server then we can compromise all of the agents." With this in mind, we try to ensure that even if the server is compromised or otherwise wants to act maliciously, the agents can protect themselves. This means they don't take arbitrary commands from the server, but rather they only take supported commands that it can map. The agent also never runs subprocesses with `SHELL=True`, which prevents shell expansion attacks and command injection attacks. The only user controlled values that go to the subprocesses are the target IP address, which must be a valid IP address to be put into the server and must be a valid IP address when the agent receives it, otherwise the agent will reject it.

Another possibile activity to consider with a compromised server is the ability to map inside the agents' local networks. Agents could be located anywhere and run by anyone, especially in a community or group oriented deployment. As such, they ship with an option called `NATLAS_SCAN_LOCAL` set to False. This will automatically reject any targets that are considered [RFC1918](https://tools.ietf.org/html/rfc1918) addresses (colloquially called 1918 addresses), thereby preventing a malicious server from enumerating the inside of the agent's network. Of course, you might _want_ to scan local addresses, such as when you're scanning your organization's `10.0.0.0/8` or similar. To support this, simply toggle `NATLAS_SCAN_LOCAL=True` for each of your agents.

There may be a future version of Natlas that worries less about this boundary, which would allow easier implementation of features such as issuing raw nmap commands or automatically distributing custom scripts. This would be ideal for single-tenant deployments where all agents are controlled by the same user. This is currently not on the roadmap, however.
