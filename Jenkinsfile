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
        stage('deploy') {
            steps {
                script {
                   echo 'Deploying Docker image to ${params.DEPLOY_ENV} environment...'
		    def ec2Instance = (params.DEPLOY_ENV == 'production') ? "ec2-user@13.39.50.25" : "ec2-user@13.39.50.50"
                    def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                   sshagent(['ec2-server-key']) {
                       sh "scp -o StrictHostKeyChecking=no server-cmds.sh ${ec2Instance}:/home/ec2-user"
                        sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                        echo "Deployment files copied successfully"
                        sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                   }
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
            subject: "Pipeline Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
                <h2>Pipeline Success</h2>
                <p>Pipeline <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> completed successfully.</p>
                <p><b>Image:</b> ${env.IMAGE_NAME}</p>
                <p>Check the build details <a href="${env.BUILD_URL}">here</a>.</p>
            """,
            to: 'your-team@example.com'
        )
    }
    failure {
        emailext(
            subject: "Pipeline Failure: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
                <h2>Pipeline Failure</h2>
                <p>Pipeline <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> failed.</p>
                <p>Please check the logs <a href="${env.BUILD_URL}">here</a>.</p>
            """,
            to: 'your-team@example.com'
        )
    }
}
}
