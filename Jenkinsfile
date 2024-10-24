#!/usr/bin/env groovy

@Library('Jenkins-shared-library')
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
                echo 'deploying the application...'
            }
        }
    }
}
