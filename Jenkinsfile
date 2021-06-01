pipeline{
    agent any

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
                container('springtest'){
                    withCredentials([usernamePassword(credentialsId: 'artifact-jenkin', usenameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
                        sh '''
                            export ARTIFACTORY_USERNAME=$USERNAME
                            export ARTIFACTORY_PASSWORD=$PASSWORD
                            ./gradlew jib
                        '''
                    }
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