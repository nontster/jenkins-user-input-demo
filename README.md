﻿# jenkins-user-input-demo
```
pipeline {
    agent {
        label "master"
    }

    stages {
        stage("Clone code from VCS") {
            steps {
                echo "Clone from Git repository"
            }
        }

        stage("Maven Build") {
            steps {
                echo "Maven Build"
            }
        }

        stage("Stage with input") {
            input {
                message "Should we release this image?"
                ok "Yes, we should."
            }
            steps {
                echo "Build number ${currentBuild.number} has approved."
            } 
        }

        stage("Publish to Nexus Repository Manager") {
            steps {
                echo "Publish to image repository"
            }
        }
    }
}
```

![Screenshot for code1](https://github.com/nontster/jenkins-user-input-demo/blob/master/Screenshot-Jenkinsfile-user-input-1.png)
