pipeline {
    agent any
    tools {
        maven 'Maven'
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
                    gv.buildJar()
                }
            }
        }

        stage("build image") {
            steps {
                script {
                    gv.buildImage()
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
