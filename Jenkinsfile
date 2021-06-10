pipeline{
    agent any

    environment{
        BUILD_TAG = "${BUILD_TIMESTAMP}"
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
                withCredentials([usernamePassword(credentialsId: 'artifact-jenkin', usenameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
                    sh '''
                        ./gradlew jibDockerBuild \
                            -Djib.to.image="devhamzah-docker.jfrog.io/springtest/springtest \
                            -Djib.to.tags="${BUILD_TAG}" \
                            -Djib.to.auth.username=$USERNAME \
                            -Djib.to.auth.password=$PASSWORD
                    '''
                }                
            }
        }

        stage("push and publish to artifacotry"){
            steps{
                rtDockerPush(
                    serverId: "jfrog-artifactory",
                    image: "devhamzah-docker.jfrog.io/springtest/springtest",
                    targetRepo: "default-docker-local",
                    buildName: "springtest",
                    buildNumber: "${BUILD_TAG}"
                )

                rtPublishInfo(
                    serverId: "jfrog-artifactory",
                    buildName: "springtest",
                    buildNumber: "${BUILD_TAG}"
                )                
            }
        }

        stage("xray scan"){
            steps{
                xrayScan(
                    serverId: "jfrog-artifactory",
                    buildName: "springtest",
                    buildNumber: "${BUILD_TAG}",
                    failBuild: true
                )                
            }
        }
    }

    post{
        always{
            cleanWs()
        }
    }
}