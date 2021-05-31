pipeline{
    agent{
        any
    }

    stages{
        stage("build"){
            steps{
                echo 'building application'
                sh './gradlew build'
            }
        }

        stage("test"){
            steps{
                echo 'Running tests'
                sh './gradlew test'
            }
        }
    }

    post{
        always{
            cleanWs()
        }
    }
}