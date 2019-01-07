---
layout: category
---

What to achieve

1. protect Jenkins File, no unauthorized change - possbile scenarios
* pull request  from non trusted - needs approval, not built
* pull request from organisation member/ collaborator -  is trusted, therefore built

1.1. pull request/fork

* currently no pull request from someone who has no write access, no fork
* fork + pull request and changed jenkins file leads to merge conflict which needs to be solved by someone trusted

2. manage User Roles in Jenkins according to Github Roles
* View Build (Log, Artefacts, Test-Results): everyone - check
* Start Build: Everyone with write access to repo - check
* Iniciate/Edit Build: Admninstrator - check
* Release Build: Everyone with write acces to repo / <s>Administrator</s> -  check

no automatic admin setup

3. run job for each pull request and commit 

4. use of shared library for jenkins pipeline configuration

Open questions
 * pull request to release/master branch? which jenkins file? 


