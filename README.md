# Demo using user input in Jenkins Pipeline
 
 ### Yes/No
 
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

 ### Input Choices 
 
 ```
 List<String> CHOICES = [];

pipeline {
    agent any

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
           steps {
              script {
                 CHOICES = ["tag1", "tag2", "tag3"];
                 env.ImageTag = input  message: 'What are we deploying today?',ok : 'Deploy',id :'tag_id',
                            parameters:[choice(choices: CHOICES, description: 'Select a tag for this build', name: 'TAG')]
                        }           
                 echo "Deploying ${env.ImageTag}."
            }
        }

        stage("Publish to Nexus Repository Manager") {
            steps {
                echo "Publish ${env.ImageTag} to image repository"
            }
        }
    }
}
 ```
 
 ![Screenshot for code2](https://github.com/nontster/jenkins-user-input-demo/blob/master/Screenshot-Jenkinsfile-input-choices-1.png)
