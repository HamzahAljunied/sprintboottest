pipeline{
    agent any

    environment{
        jFrogCredential = credentials('jfrog-jenkins')
    }

    stages{
        stage("build"){
            steps{
                echo 'building application'
                sh ' chmod +x ./gradlew'
                sh './gradlew build'
            }
        }

        stage("test"){
            steps{
                echo 'Running tests'
                sh ' chmod +x ./gradlew'
                sh './gradlew test'
            }
        }

        stage("Build & Push to artifactory"){
            steps{
                echo 'building image'
                sh 'chmod +x ./gradlew'
                sh './gradlew jib \
                    -Djib.to.tags=${BUILD_TIMESTAMP} \
                    -Djib.to.auth.username=$jFrogCredential_USR \
                    -Djib.to.auth.password=$jFrogCredential_PSW'
            }
        }
    }

    post{
        always{
            cleanWs()
        }
    }
}