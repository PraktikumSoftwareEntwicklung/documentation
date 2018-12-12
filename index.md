---
layout: category
---

What to achieve

1. protect Jenkins File, no unauthorized change - possbile scenarios
* pull request  from non trusted
* pull request from organisation member/ collaborator
* comit on branch from non trusted
* comit on branch from organisation member/ collaborator

currently no pull request from someone who has no write access

2. manage User Roles in Jenkins according to Github Roles
* View Build (Log, Artefacts, Test-Results): everyone - check
* Start Build: Everyone with write access to repo - check
* Iniciate/Edit Build: Admninstrator - check
* Release Build: Everyone with write acces to repo / <s>Administrator</s> -  check

3. run job for each pull request and commit 

4. use of shared library for jenkins pipeline configuration
