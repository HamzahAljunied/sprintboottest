pipeline{
    agent{
        any
    }

    stages{
        stage("build"){
            echo 'building application'
            sh './gradlew build'
        }

        stage("test"){
            echo 'Running tests'
            sh './gradlew test'
        }
    }

    post{
        always{
            cleanWs()
        }
    }
}