#!/usr/bin/env groovy

pipeline {
    agent any
    stages {
        stage('build app') {
            steps {
               script {
                   echo "building the application..."
               }
            }
        }
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                }
            }
        }
        stage('deploy to k8s') {
            environment {           // this whole env block isn't needed for linode LKE or bare metal for ex, bec we had a credential that saved kubeconfig as secret file in jenkins ui
                AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id') 
                AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
            }
            steps {
                script {
                   echo 'deploying image to k8s...'
                   //withKubeConfig(credentialsId: 'lke-credentials', serverUrl: '<my linode LKE endpoint>') {
                        sh 'kubectl create deployment nginx-deployment --image=nginx'
                    }
                }
            }
        }
    }
}
