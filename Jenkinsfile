pipeline{
    agent{
        kubernetes{
            yaml"""
                apiVersion: v1
                kind: Pod
                metadata:
                    labels:
                        jenkins/label: jenkins-slave
                spec:
                    containers:
                    - name: jgc
                      image: dsingh107/jenkins-agent:latest
                      imagePullPolicy: Always
                      tty: true
                      volumeMounts:
                      - name: docker
                        mountPath: /var/run/docker.sock
                    volumes:
                        - name: docker
                          hostPath:
                            path: /var/run/docker.sock
            """
        }
    }

    environment{
        BUILD_TAG = "${BUILD_TIMESTAMP}"
    }

    stages{
        stage("build"){
            steps{
                container('jgc'){
                    echo 'building application'
                    sh ' chmod +x ./gradlew'
                    sh './gradlew build'
                }
            }
        }

        stage("test"){
            steps{
                container('jgc'){
                    echo 'Running tests'
                    sh ' chmod +x ./gradlew'
                    sh './gradlew test'
                }
            }
        }

        stage("Build & Push to artifactory"){
            steps{
                echo 'building image'
                sh './gradlew jib'
                container('jgc'){
                    withCredentials([usernamePassword(credentialsId: 'artifact-jenkin', usenameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
                        sh '''
                            export ARTIFACTORY_USERNAME=$USERNAME
                            export ARTIFACTORY_PASSWORD=$PASSWORD
                            ./gradlew jibDockerBuild \
                                -Djib.to.image="devhamzah-docker.jfrog.io/springtest/springtest \
                                -Djib.to.tags="${BUILD_TAG}" \
                                -Djib.to.auth.username=$USERNAME \
                                -Djib.to.auth.password=$PASSWORD
                        '''
                    }
                }
            }
        }

        stage("push and publish to artifacotry"){
            steps{
                container('jgc'){
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
        }

        stage("xray scan"){
            steps{
                container('jgc'){
                    xrayScan(
                        serverId: "jfrog-artifactory",
                        buildName: "springtest",
                        buildNumber: "${BUILD_TAG}",
                        failBuild: true
                    )
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