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
            steps {
                script {
                   echo 'deploying image to k8s...'
                   withKubeConfig(clusterName: 'demo-cluster-2', credentialsId: 'K8S', serverUrl: 'https://5A7958397024D2B4E979915A7FEBDC6A.gr7.us-east-1.eks.amazonaws.com') {
                        sh 'kubectl create deployment nginx-deployment --image=nginx'
                    }
                }
            }
        }
    }
}
