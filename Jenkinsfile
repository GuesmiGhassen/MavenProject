pipeline {
    agent any
    tools {
        maven 'maven-3.9'
    }
    stages {
        stage("build jar") {
            steps {
                script{
                   echo "Building the app..."
                    sh 'mvn package' 
                }
            }
        }
        stage("build image"){
            steps {
                script{
                  echo "Building the image..."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'Pass', usernameVariable: 'User')]){
                        sh 'docker build -t ghassen07/demo-app:1.0 .'
                        sh "echo $Pass | docker login -u $User --password-stdin"
                        sh 'docker push ghassen07/demo-app:1.0'
                    }  
                }
                
            }
        }
        stage("deploy") {
            steps {
                script{
                    echo "Deploying the app..."
                }
            }
        }
    }
}
