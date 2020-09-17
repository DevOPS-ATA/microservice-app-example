#!groovy

pipeline {
    agent {
        kubernetes {
            defaultContainer 'jnlp'
            yaml """
                    apiVersion: v1
                    kind: Pod
                    metadata:
                    labels:
                    spec:
                      imagePullSecrets:
                      - name: regcred-webpre                                  
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
                        imagePullPolicy: IfNotPresent
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
        stage("Deploy to Nexus") {
            steps {
                container('maven') {
                    script {
                        ws("$WORKSPACE/users-api/") {
                            sh 'cat /root/.m2/settings.xml'
                            sh 'mvn deploy -Dmaven.test.skip=true'
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
