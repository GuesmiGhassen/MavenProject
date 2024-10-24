#!/usr/bin/env groovy

@Library('Jenkins-shared-library')_
def gv

pipeline {
    agent any
    tools {
        maven 'maven-3.9'
    }
    stages {
        stage("init"){
            steps {
                script {
                    gv = load "script.groovy"
                }
            }
        }
        stage("test") {
            steps {
                echo 'testing the application...'
                echo "executing pipeline for $BRANCH_NAME..."
            }
        }
        
        stage("build jar") {
            steps {
                script {
                    buildJar()
                }
            }
        }

        stage("build image") {
            steps {
                script {
                    buildImage()
                }
            }
        }

        stage("deploy") {
            when {
                expression {
                    BRANCH_NAME == 'main'
                }
            }
            steps {
                script {
                    def dockerCMD = 'docker run -p 3080:3080 -d ghassen07/my-repo:1.0'
                    sshagent(['ec2-server-key']) {
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@35.181.50.177 ${dockerCMD}"
                    }
                }
            }
        }
    }
}
