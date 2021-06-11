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

        stage("build image"){
            steps{
                container('jgc'){
                    echo 'building image'
                    withCredentials([usernamePassword(credentialsId: 'jfrog-hamzah-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
                        sh '''
                            ./gradlew jibDockerBuild \
                                -Djib.to.image="devhamzah.jfrog.io/default-docker-local/springtest" \
                                -Djib.to.tags="${BUILD_TAG}" \
                                -Djib.to.auth.username=$USERNAME \
                                -Djib.to.auth.password=$PASSWORD
                        '''
                    }
                }
            }
        }

        stage("push and publish to artifactory"){
            steps{
                container('jgc'){
                    rtDockerPush(
                        serverId: "jfrog-hamzah",
                        image: "devhamzah.jfrog.io/default-docker-local/springtest",
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
