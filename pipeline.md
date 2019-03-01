---
layout: category
title: Create the Pipeline Definition
---

## Definitions with security background

### Protection of the Maven .m2-Cache

To speed up the build process it is wanted to save the cache which maven creates during the build, so that in the next build not every dependency has to be downloaded again. But it is needed to protect this cache, so no possible attacker could modify the cache e.g. in a pull request to inject unwanted code.
To realize this protection only builds on the master branch have write access to the saved cache. Builds on all other branches and on all pull requests should only be able to read the saved cache. So first of all we need a distinction between builds on the master branch and all other builds:

```Jenkinsfile
pipeline {
    stages {
        stage ('Build_Master') {
            agent {
                ...
            }
            when {
                expression {
                    if (env.CHANGE_TARGET) {    // check pull request
                        return false
                    }
                    return env.GIT_BRANCH == 'master'
                }
            }
            stages {
                // stages for the master build
            }
        }
        stage('Build_Slave') {
            ...
            when {
                expression {
                    if (env.CHANGE_TARGET) {
                        return true
                    }
                    return !(env.GIT_BRANCH == 'master')
                }
            }
            ...
        }
    }
}
```

To make the saved cache persistant, we need a folder outside of the running container on the server. For more information on how to access files outside of a container see ['Setup Dockercontainer & Jenkins'](docker). Here again we differentiate between master and all other builds, which only get read-only access to the folder:

```Jenkinsfile
...
stage('Build_Master') {
    agent {
        docker {
            image 'custom_maven:latest'
            args "... -v /media/data/m2-cache/:/home/jenkinsbuild/tmp_cache ..."
        }
    }
    ...
}
stage('Build_Slave') {
    agent {
        docker {
            image 'custom_maven:latest'
            args '... -v /media/data/m2-cache/:/home/jenkinsbuild/tmp_cache:ro ...'
        }
    }
    ...
}
...
```

If that is done we need to access the saved cache. Since the mounted folder is read-only in the not-master case, we need to copy the content of the folder to the location of the actual build (before the build runs). And in the master case we also need to copy the generated cache into the mounted folder (after the build):

```Jenkinsfile
stage('load_cache') {
    steps {
        sh 'mkdir /home/jenkinsbuild/.m2/'
        sh 'cp -r /home/jenkinsbuild/tmp_cache/. /home/jenkinsbuild/.m2/'
    }
}

// only in master builds:
stage('save_cache') {
    steps {
        sh 'cp -r /home/jenkinsbuild/.m2/. /home/jenkinsbuild/tmp_cache/'
    }
}
```

