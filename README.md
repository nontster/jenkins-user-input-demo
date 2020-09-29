# Demo using user input in Jenkins Pipeline
 
### Docker image promotion - Copy image from Non-Production to Production Docker Registry

[Image Tag Parameter Plug-in](https://plugins.jenkins.io/image-tag-parameter/) 


```
def SITE_NON_PROD = 'registry-1.docker.io'
def SITE_PROD = 'nexus:5000'

pipeline {
  agent any
  
  parameters {
    imageTag(name: 'DOCKER_IMAGE', description: '',
             image: 'jenkins/jenkins', filter: '.*', defaultTag: '',
             registry: '', credentialId: '', tagOrder: 'NATURAL')
  }

  stages {
    stage('Pull image from Non-Prod Registry') {
        steps {
            echo "${env.DOCKER_IMAGE}" // will print selected image name with tag (eg. jenkins/jenkins:lts-jdk11)
            echo "${env.DOCKER_IMAGE_TAG}" // will print selected tag value (eg. lts-jdk11)
            echo "${env.DOCKER_IMAGE_IMAGE}" // will print selected image name value (eg. jenkins/jenkins)
        
            script {
                docker.withRegistry("https://${SITE_NON_PROD}",'non-prod-registry-credentials') {
                    sh "docker pull ${env.DOCKER_IMAGE}"
                }
            }
        }
    }
    
    stage('Re-Tag image') {
         steps {
             script {
                sh "docker tag ${env.DOCKER_IMAGE} ${SITE_PROD}/${env.DOCKER_IMAGE}"
             }      
         }
    }
    
    stage('Push image to Prod') {
         steps {
             script {
                docker.withRegistry("http://${SITE_PROD}",'prod-registry-credentials') {
                    sh "docker push ${SITE_PROD}/${env.DOCKER_IMAGE}"
                }
             }      
         }
    }
    
    stage('Cleanup') {
         steps {
             script {
                sh "docker rmi ${env.DOCKER_IMAGE}" 
                sh "docker rmi ${SITE_PROD}/${env.DOCKER_IMAGE}"
             }      
         }
    }
  }
}
```
![Screenshot for code1](https://github.com/nontster/jenkins-user-input-demo/blob/master/images/Screenshot-copy-image-from-nonprd-to-prd.png)

___

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

![Screenshot for code1](https://github.com/nontster/jenkins-user-input-demo/blob/master/images/Screenshot-Jenkinsfile-user-input-1.png)

___

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

 ![Screenshot for code2](https://github.com/nontster/jenkins-user-input-demo/blob/master/images/Screenshot-Jenkinsfile-input-choices-1.png)
