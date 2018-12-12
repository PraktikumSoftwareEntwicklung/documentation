---
layout: category
---

What to achieve

1. protect Jenkins File, no unauthorized change - possbile scenarios
* pull request  from non trusted
* pull request from organisation member/ collaborator
* comit on branch from non trusted
* comit on branch from organisation member/ collaborator

2. manage User Roles in Jenkins according to Github Roles

  § Github Owner is Jenkins Administrator

  §  View Build (Log, Artefacts, Test-Results): everyone
  
  §  Start Build: Everyone with write access to repo
  
  §  Edit/ Creat Build-Job: Administrator
  
  §  Release Build: Everyone with write acces to repo / Administrator 

3. run job for each pull request and commit 
4. use of shared library for jenkins pipeline configuration
