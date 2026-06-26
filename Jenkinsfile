#!/usr/bin/env groovy

// library identifier: 'jenkins-shared-library@main', retriever: modernSCM(
//     [$class: 'GitSCMSource',
//     remote: 'https://github.com/lpadgett23/jenkins-shared-library.git',
//     credentialsID: 'gitlab-credentials'
//     ]
// )

pipeline {
    agent any
    tools {
        maven 'maven-3.9'
    }
    environment {
        DOCKER_REPO_SERVER = '032080729375.dkr.ecr.us-east-1.amazonaws.com'
        DOCKER_REPO = '${DOCKER_REPO_SERVER}/java-maven-app'
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing version of app...'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "aws-$version-$BUILD_NUMBER"
                }
            }
        }
        stage('build app'){
            steps {
                script {
                    echo 'building th application ... '
                    sh 'mvn clean package'
                }
            }
        }
        stage("build image") {
            steps {
                script {
                    echo "building the docker image..."
                    //withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]){
                    withCredentials([usernamePassword(credentialsId: 'ecr-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]){
                        sh "docker build -t ${DOCKER_REPO}:${IMAGE_NAME} ."
                        sh 'echo $PASS | docker login -u $USER --password-stdin ${DOCKER_REPO_SERVER}'
                        sh "docker push ${DOCKER_REPO}:${IMAGE_NAME}"
                    }
                }
            }
        }

        stage('deploy to k8s') {
            environment {          
                AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id') 
                AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
                APP_NAME = 'java-maven-app'
            }
            steps {
                script {
                    echo 'deploying docker image to k8s...'
                    sh 'envsubst < kubernetes/deployment.yaml | kubectl apply -f -'
                    sh 'envsubst < kubernetes/service.yaml | kubectl apply -f -'
                }
            }
        } 

        stage("commit version update") {
            steps {
                script {
                    //withCredentials([string(credentialsId: 'github-jenkins-token', variable: 'TOKEN'), 
                    withCredentials([usernamePassword(credentialsId: 'github-jenkins-token', passwordVariable: 'PASS', usernameVariable: 'USER')]){
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "Jenkins"'
                        sh 'git status'
                        sh 'git branch'
                        sh 'git config --list'
                        sh "git remote set-url origin https://${USER}:${PASS}@github.com/lpadgett23/11-eks-kubernetes-on-aws.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh "git push origin HEAD:jenkins-jobs"  // this used to have /folder-name at end... 
                    }   
                }
            }               
        }             
    }
} 
