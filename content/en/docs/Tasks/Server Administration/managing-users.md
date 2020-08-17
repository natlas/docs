---
title: "Managing Users"
linkTitle: "Managing Users"
date: 2017-01-05
weight: 1
description: >
  Learn about the tools available to manage your users
---

## Adding Your First User

Your first user is the most important for your Natlas server. You'll want to create yourself an admin account so that you can manage your deployment. The first user must be [invited via cli](#inviting-by-cli). All container references for CLI assume that your container is named `natlas_server`.

## Inviting New Users

### Inviting by CLI

Inviting via CLI is pretty easy but has one small quirk - you have to ensure the `SERVER_NAME` environment variable is set for your server so that invitation links can be generated correctly. If it's not set, the command will fail.

```bash
$ docker exec -e SERVER_NAME=example.com -it natlas_server flask user new --admin
Accept invitation: http://example.com/auth/invite?token=this-is-an-invalid-example-token
```

You can then take this invitation into your browser and create your account.

### Inviting by Web

If you already have an account, inviting a new user via the web interface is also very easy at `/admin/users`. Simply add their email address in the text box and hit Invite User.

![Invite Users](https://i.imgur.com/y8a1AD4.png)

## Promoting A User

Promoting a user means to change their role from a normal user to an admin.

### Promoting by CLI

Promoting a user via CLI requires two things: Access to the docker container and the email address of the user you want to promote.

```bash
$ docker exec -it natlas_server flask user role --promote user@example.com
user@example.com is now an admin
```

### Promoting by Web

Promoting a user on the web panel is super easy as long as you already have an admin account. Simply find their email address in the `/admin/users` page and click the Promote button.

![Promoting a user](https://i.imgur.com/1HRYXUW.png)
