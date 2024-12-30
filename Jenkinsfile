#!/usr/bin/env groovy

library identifier: 'Jenkins-shared-library@main', retriever: modernSCM(
    [$class: 'GitSCMSource',
     remote: 'https://github.com/GuesmiGhassen/Jenkins-shared-library.git',
     credentialsId: 'Github-Credentials'
    ]
)

pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['staging', 'production'], description: 'Deployment environment')
        string(name: 'VERSION', defaultValue: 'latest', description: 'Application version to deploy')
    }
    stages {
	    stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
	    stage('test') {
            steps {
                script {
                    echo 'Running tests...'
                    sh 'mvn test'
                }
            }
        }
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
		   echo "Image name: ${env.IMAGE_NAME}" 
                   buildDockerImage(env.IMAGE_NAME)
                   dockerLogin()
                   dockerPush(env.IMAGE_NAME)
                }
            }
        }
        stage('deploy via EKS') {
            environment {
                AWS_ACCESS_KEY_ID = credentials('aws_access_key_id')
                AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
                APP_NAME = 'java-maven-app'
            }
            steps {
                script {
                    echo 'deploying docker image...'
                    sh 'envsubst < kubernetes/deployment.yaml | kubectl apply -f -'
                    sh 'envsubst < kubernetes/service.yaml | kubectl apply -f -'
                }
            }
        }
        stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'Github', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        // git config here for the first time run
                        sh 'git config --global user.email "ghassengasmi34@gmail.com"'
                        sh 'git config --global user.name "GuesmiGhassen"'

                        sh "git remote set-url origin https://${USER}:${PASS}@github.com/GuesmiGhassen/MavenProject.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:main'
                    }
                }
            }
        }
	    stage('cleanup') {
            steps {
                script {
                    echo 'Cleaning up unused Docker resources...'
                    sh 'docker system prune -f'
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline execution completed.'
        }
        success {
            emailext(
                subject: "Pipeline Success: ${env.IMAGE_NAME}",
                body: """
                    <h2>Pipeline Success</h2>
                    <p><b>Image:</b> ${env.IMAGE_NAME}</p>
                """,
                to: 'ghassengasmi34@gmail.com'
            )
        }
        failure {
            emailext(
                subject: "Pipeline Failure: ${env.IMAGE_NAME}",
                body: """
                    <h2>Pipeline Failure</h2>
                    <p><b>Image:</b> ${env.IMAGE_NAME}</p>
                """,
                to: 'ghassengasmi34@gmail.com'
            )
        }
    }
}
