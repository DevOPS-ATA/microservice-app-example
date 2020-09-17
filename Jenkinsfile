#!groovy

pipeline {
    agent {
        kubernetes {
            defaultContainer 'jnlp'
            yaml """
                    apiVersion: v1
                    kind: Pod
                    metadata:
                      annotations:
                        openshift.io/scc: privileged
                    labels:
                    spec:
                      imagePullSecrets:
                      - name: deployer-dockercfg-bxmwv
                      serviceAccountName: pipeline                                 
                      containers:
                      - name: maven
                        image: maven:3-jdk-8-slim
                        workingDir: /home/jenkins/agent
                        imagePullPolicy: Always   
                        volumeMounts:
                        - name: settings-xml
                          mountPath: /usr/share/maven/ref/
                        - name: volume-ca-bundle
                          mountPath: /etc/ssl/certs/java/cacerts # Alpine / Debian / Ubuntu / Gentoo etc.
                          subPath: cacerts                                  
                        command:
                        - cat
                        tty: true
                      - name: buildah
                        image: 'quay.io/buildah/stable:v1.11.0'
                        imagePullPolicy: Always
                        command:
                        - /bin/cat
                        tty: true
                        securityContext:
                          privileged: true                        
                        volumeMounts:
                        - mountPath: /var/lib/containers
                          name: buildah-storage                        
                      volumes:
                      - name: settings-xml
                        configMap:
                          name: settings-xml
                      - name: volume-ca-bundle
                        configMap:
                          name: ca-bundle
                      - name: buildah-storage
                        emptyDir: {}                         
                """
        }
    }

    options {
        timeout(time: 2, unit: 'HOURS')
        disableConcurrentBuilds()
    }

    stages {
        stage("Prueba") {
            steps {
                container("buildah") {
                    script {
                        sh 'buildah images'
                    }
                }
            }
        }

        stage("Build") {
            steps {
                container("maven") {
                    script {
                        sh 'ls -la'
                        ws("$WORKSPACE/users-api/") {
                            sh 'mkdir /root/.m2/'
                            sh 'cp /usr/share/maven/ref/settings.xml /root/.m2/settings.xml'
                            sh 'mvn clean compile'
                        }
                    }
                }
            }
        }
        stage("Test") {
            steps {
                container("maven") {
                    ws("$WORKSPACE/users-api/") {
                        sh 'mvn verify'
                    }
                }
            }
        }

        stage("Package") {
            steps {
                container("maven") {
                    script {
                        ws("$WORKSPACE/users-api/") {
                            sh 'mvn package'
                        }
                    }
                }
            }
        }
        stage("Upload artifac to Nexus") {
            steps {
                container('maven') {
                    script {
                        ws("$WORKSPACE/users-api/") {
                            sh 'mvn deploy -Dmaven.test.skip=true'
                        }
                    }
                }
            }
        }
        stage("Create and Upload image to Registry") {
            steps {
                container('buildah') {
                    script {
                        ws("$WORKSPACE/users-api/") {
                            sh 'export TOKEN=eyJhbGciOiJSUzI1NiIsImtpZCI6Ikc3OExpRURGZTZSNHJTWEhPV1VaS0FXZmRObllPcU96NldzekV4V0taMFEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6InBpcGVsaW5lLXRva2VuLXM2ZDQyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6InBpcGVsaW5lIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMjVhMzYzMWYtYzc4OC00NzdlLTgzZDAtZWI3ZTJiOGI1YWQyIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRlZmF1bHQ6cGlwZWxpbmUifQ.eBfaD7PKq59flvae-v-TFqxLIjjm7pxDOhlsRIBblItUnii6NEo5TiWFz9Hfd8wW63Zi9YdwYWer9zFpdJeUyFFL1zYmItYLEgqXFqdiF9iafNoUvMtZGGdn9QBK3VKyCbOyvNGuWzctAwMEtiTgkEFhbFv_DoYFtLJnJ5rNWxjXjnI71L9firczGSSqehf1bX43RdKuXbDt1ka-8AXEfDvn1fKi169xu4EjS6YcEFFEqLLYzB_Kzlq8s45oW7QpDnVmKle6q9HaRv-wo485fFtcNNP36nwREzfGCyQOxd3IDlvuWG7M7Mc0ayTEepf4CXThnCOQy5EniR5xO0vk1Q'
                            sh 'echo $TOKEN | buildah login -u sa --password-stdin image-registry.openshift-image-registry.svc:5000'
                            sh "buildah bud --format=oci --tls-verify=true --layers -f ./Dockerfile -t image-registry.openshift-image-registry.svc:5000/microapp/userapi:master ."
                            sh "buildah push --tls-verify=false  image-registry.openshift-image-registry.svc:5000/microapp/userapi:master docker://image-registry.openshift-image-registry.svc:5000/microapp/userapi:master"
                            sh 'buildah images'
                        }
                    }
                }
            }
        }
    }

//    post {
//        success {
//            script {
//                maven.PostSuccess()
//                slackReports.AggregateMessage(true)
//            }
//
//        }
//        failure {
//            script {
//                slackReports.AggregateMessage(false)
//            }
//        }
//        always {
//            cleanWs()
//        }
//    }
}
