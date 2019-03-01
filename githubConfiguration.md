---
layout: category
title: Github Configuration
sidebar_sort_order: 5
---
## Enabling Oauth Login

Go to the Github Page in settings > OAuth Apps > New OAuth App'. The values for application name, homepage URL, or application description don't matter. They can be customized however desired.

However, the authorization callback URL takes a specific value. It must be https://jenkins.example.com/securityRealm/finishLogin where jenkins.example.com is the location of the Jenkins server.           

The important part of the callback URL is /securityRealm/finishLogin
Finish by clicking Register application and you are done.

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
