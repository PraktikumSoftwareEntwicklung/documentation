---
layout: category
title: Project Goals
---



1. protect Jenkins File, no unauthorized change 
* pull request  from non trusted - needs approval, not built
* pull request from organisation member/ collaborator -  is trusted, therefore built

2. manage User Roles in Jenkins according to Github Roles
* View Build (Log, Artefacts, Test-Results): everyone - check
* Start Build: Everyone with write access to repo - check
* Iniciate/Edit Build: Admninstrator - check
* Release Build: Everyone with write acces to repo / <s>Administrator</s> -  check

3. run job for each pull request and commit 

4. use of shared library for jenkins pipeline configuration

5. Protect built by running in a docker container

Open questions
 * pull request to release/master branch? which jenkins file? 


