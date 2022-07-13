// prerequisites: a nodejs app must be deployed inside a kubernetes cluster
//                repo must include the Dockerfile
//                docker pipeline must be installed on Jenkins
//                docker credentials must be created on Jenkins
// TODO: look for all instances of [] and replace all instances of 
//       '[VARIABLE]' with actual values 
//        e.g https://github.com/rpnayak80/external_events.git might become https://github.com/MyName/external.git
// variables:
//      roidtc-july2022-user203
//      docker  //id of your global docker credentials in Jenkins https://www.google.com/search?q=add+docker+credentials+Jenkins&oq=add+docker+credentials+Jenkins
//      https://github.com/rpnayak80/external_events.git
//      raghav
//      external
//      cluster-1 
//      us-central1-c. //THIS NEEDS TO CHANGE FOR THE AWS VERSION
//      default
//      the following values can be found in the yaml:
//      demo-ui-deployment
//      demo-ui-container (name of the container to be replaced - in the template/spec section of the deployment)


pipeline {
    agent any 
   environment {
        registryCredential = 'raghav'
        imageName = 'rpnayak80/external_events'
        dockerImage = ''
        }
    stages {
        stage('Run the tests') {
             agent {
                docker { 
                    image 'node:14-alpine'
                    args '-e HOME=/tmp -e NPM_CONFIG_PREFIX=/tmp/.npm'
                    reuseNode true
                }
            }
            steps {
                echo 'Retrieving source from github' 
                git branch: 'main',
                    url: 'https://github.com/rpnayak80/external_events.git'
                echo 'Did we get the source?' 
                sh 'ls -a'
                echo 'install dependencies' 
                sh 'npm install'
                echo 'Run tests'
                sh 'npm test'
                echo 'Tests passed on to build and deploy Docker container'
            }
        }
        stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build imageName
                }
            }
            }
            stage('Deploy Image') {
            steps{
                script {
                docker.withRegistry( '', registryCredential ) {
                    dockerImage.push("$BUILD_NUMBER")
                }
                }
            }
        }     
         stage('get k8s credentials') {
             agent {
                docker { 
                    image 'google/cloud-sdk:latest'
                    args "-e HOME=${WORKSPACE}/kube/"
                    reuseNode true
                    }
                    }
            steps {
                sh "rm -rf ${WORKSPACE}/kube/"
                echo 'Get cluster credentials'
                sh 'gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project roidtc-july2022-user203'
            }
        }     
         stage('update k8s') {
             agent {
                docker { 
                    image 'google/cloud-sdk:latest'
                    args '-e HOME=/tmp'
                    reuseNode true
                        }
                    }
            steps {
                echo 'Set the image'
                     sh "kubectl --kubeconfig=${WORKSPACE}/kube/.kube/config set image deployment/demo-ui-deployment demo-ui-container=${env.imageName}:${env.BUILD_NUMBER}"
            }
        }     
        stage('Remove local docker image') {
            steps{
                sh "docker rmi $imageName:latest"
                sh "docker rmi $imageName:$BUILD_NUMBER"
            }
        }
    }
}
