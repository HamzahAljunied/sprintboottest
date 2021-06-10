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
                withCredentials([usernamePassword(credentialsId: 'jfrog-hamzah-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
                    sh '''
                        docker load
                        docker image ls -a
                        ./gradlew jibDockerBuild \
                            -Djib.to.image="devhamzah-docker.jfrog.io/springtest/springtest" \
                            -Djib.to.tags="${BUILD_TAG}" \
                            -Djib.to.auth.username=$USERNAME \
                            -Djib.to.auth.password=$PASSWORD
                    '''
                }                
            }
        }

        stage("push and publish to artifactory"){
            steps{
                rtDockerPush(
                    serverId: "jfrog-hamzah",
                    image: "devhamzah-docker.jfrog.io/springtest/springtest",
                    targetRepo: "default-docker-local",
                    buildName: "springtest",
                    buildNumber: "${BUILD_TAG}"
                )

                rtPublishInfo(
                    serverId: "jfrog-hamzah",
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
