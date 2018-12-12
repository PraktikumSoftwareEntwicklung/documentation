---
layout: category
---

What to achieve

1. protect Jenkins File, no unauthorized change - via plugin?
* pull request to master from collaborator - gets jenkinsfile from master branch
* comit on branch from collaborator/owner - gets jenkinsfile from branch
* pull request no comiter - build does not get triggered, jenkinsfile is protected, only trusted from admins
2. manage User Roles in Jenkins according to Github Roles

  § Github Owner is Jenkins Administrator

  §  View Build (Log, Artefacts, Test-Results): everyone
  
  §  Start Build: Everyone with write access to repo
  
  §  Edit/ Creat Build-Job: Administrator
  
  §  Release Build: Everyone with write acces to repo / Administrator 

3. run job for each pull request and commit 
4. use of shared library for jenkins pipeline configuration
