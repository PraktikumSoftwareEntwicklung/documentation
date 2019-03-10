---
layout: category
title: Setup Instructions
sidebar_sort_order: 15
---

## Setup instructions

1. Basic setup like described in [Basic System Setup](basicSystemSetup.md)  
  1.1. Create partitions  
  1.2. Install docker  
  1.3. Configure container (after step 2)   
2. Create docker container like described in [Setup Dockercontainer & Jenkins](setupDockercontainerJenkins.md)  
  2.1. Create docker images from the provided dockerfiles  
  2.2. Create docker network for use of proxy  
  2.3. Create folder and set permissions for user 1500 for the cache and the buildfiles (set quota if needed)  
  2.4. Add volume mounts for the jenkins container \[1\]  
  2.5. Other jenkins container config adaptions \[2\]  
  2.6. Start container

\[1\]:
* /media/data/jenkins:/var/jenkins_home  
* /var/run/docker.sock:/var/run/docker.sock  
* /media/docker2:/var/buildfiles  

\[2\]:
* User set to 1500  
* New image name  
