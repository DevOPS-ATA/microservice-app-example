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
                        image: maven:3-jdk-11-slim
                        workingDir: /home/jenkins/agent
                        imagePullPolicy: Always   
                        volumeMounts:
                        - name: settings-xml
                          mountPath: /usr/share/maven/ref/settings.xml                                  
                        command:
                        - cat
                        tty: true
                      volumes:
                      - name: settings-xml
                        configMap:
                          name: settings-xml
                """
        }
    }

    options {
        timeout(time: 2, unit: 'HOURS')
        disableConcurrentBuilds()
    }

    stages {

        stage("Build") {
            steps {
                container("maven") {
                    script {
                        sh 'ls -la'
                        ws("$WORKSPACE/users-api/") {
                            sh 'ls -la'
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
                            sh 'mvn deploy'
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
