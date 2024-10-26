#!/usr/bin/env groovy

library identifier: 'Jenkins-shared-library@main', retriever: modernSCM(
    [$class: 'GitSCMSource',
     remote: 'https://github.com/GuesmiGhassen/Jenkins-shared-library.git',
     credentialsId: 'Github'
    ]
)

pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        IMAGE_NAME = 'ghassen07/my-repo:2.0'
    }
    stages {
        stage('build app') {
            steps {
               script {
                  echo 'building application jar...'
                  buildJar()
               }
            }
        }
        stage('build image') {
            steps {
                script {
                   echo 'building docker image...'
                   buildImage(env.IMAGE_NAME)
                   dockerLogin()
                   dockerPush(env.IMAGE_NAME)
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                   echo 'deploying docker image to EC2...'
                    def dockerComposeCmd = "docker-compose -f docker-compose.yaml up --detach"
                   sshagent(['ec2-server-key']) {
                        sh "scp docker-compose.yaml ec2-user@13.39.50.25:/home/ec2-user"
                        echo "Copied successfully"
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@13.39.50.25 ${dockerComposeCmd}"
                   }
                }
            }
        }
    }
}
