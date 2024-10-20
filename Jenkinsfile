pipeline {
    agent any

    stages {
        stage("test") {
            steps {
                echo 'testing the application...'
                echo "executing pipeline for $BRANCH_NAME..."
            }
        }
        
        stage("build") {
            when {
                expression {
                    BRANCH_NAME == 'main'
                }
            }
            steps {
                echo 'building the application...'
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
