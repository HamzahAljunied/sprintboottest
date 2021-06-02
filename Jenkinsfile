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
                withCredentials([usernamePassword(credentialsId: 'jfrog-jenkins', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
                    sh 'chmod +x ./gradlew'
                    sh './gradlew jib \
                        -Djib.to.tags=latest \
                        -Djib.to.auth.username=$USERNAME \
                        -Djib.to.auth.password=$PASSWORD'
                }
            }
        }
    }

    post{
        always{
            cleanWs()
        }
    }
}