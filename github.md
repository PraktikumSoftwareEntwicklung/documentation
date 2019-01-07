---
layout: category
title: Github Configuration
---

## Webhook

To let Jenkins know of a new pull requets or commit you need to setup a webhook. 
In your Organization go to Settings > Webhooks > Add webhook.
Add a webhook with the the Payload URL of your Jenkins Instance and leave settings as they are.

## Branch Protection in a repository

Go to the main page of the repository > Settings > Branches > Add Rule.
Apply the following.
* Apply rule to: master
* Require pull request reviews before merging 
* Dismiss stale pull request approvals when new commits are pushed
* Include administrators 
