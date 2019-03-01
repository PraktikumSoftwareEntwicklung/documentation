---
layout: index
title : Continous Integration
---

This website is dedicated to describe a continous integration concept for an Open Source project using Github and Jenkins. Each commit and pull request is being build, the status is visible in Github and can be verified immediatly.

The goal is to have all the files and build logs publicy available but protect the build jobs from corruption. The build jobs are being build in a Docker container to prevent any damage to the Jenkins instance. 

The Jenkins file itself is in the Github repository and is protected from unauthorized change and usage from non trusted users.



| Attack Scenario     | Protection |
| -----------      | ----------- |
| memory overflow     | cache is filled once and then ready only     |
| server overload  | limitation of docker container and timeout       |
| sensitive data (keys, passwords, ...)  | protected by jenkins        |
| unportected access to docker container  | container does not have root rights and file system is not mounted  |
| corrupt m2 cache  | cache is filled once and then ready only and seperate cache for each repo   |
| change build coniguration  | protected by jenkins        |
