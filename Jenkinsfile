#!groovy

pipeline {
    agent {
        kubernetes {
            label 'microservices-pipeline'
            defaultContainer 'jnlp'
            idleMinutes '5'
            yaml """
                    apiVersion: v1
                    kind: Pod
                    metadata:
                    labels:
                    spec:
                      imagePullSecrets:
                      - name: regcred-webpre
                      nodeSelector:
                        jenkins-slave: "true"
                      containers:
                      - name: maven
                        image: webpre-adm.es.sedc.internal.vodafone.com:44150/devops/vf-maven-slave:3.6-jdk-8-slim
                        volumeMounts:
                        - name: maven-settings
                          mountPath: /usr/share/maven/conf/settings.xml
                          subPath: settings.xml
                        workingDir: /home/jenkins/agent
                        imagePullPolicy: Always
                        command:
                        - cat
                        tty: true
                """
        }
    }

    options {
        timeout(time: 2, unit: 'HOURS')
        disableConcurrentBuilds()
    }

    stages {

        stage("Initialize Steps Tools") {
            steps {
                container("jnlp") {
                    script {
                        environment.ConfigVariables()
                        gitUtils.getLastCommitterToSlack([branch: "${env.GIT_BRANCH}"])

                    }
                }
            }
        }

        stage("Build") {
            steps {
                container("maven") {
                    script {
                        mavenArtifacts = maven.resolveMavenArtifacts(context)
                        maven.Build()
                    }
                }
            }
        }
        stage("Test") {
            steps {
                container("maven") {
                    script {
                        maven.Test()

                    }
                }
            }
        }
        stage("JaCoCo") {
            when {
                expression { return !(env.branch ==~ /(?i)(PR-master)/) }
            }
            steps {
                container("maven") {
                    script {
                        maven.Jacoco()
                    }
                }
            }
        }

        stage("Package") {
            when {
                expression { return !(env.branch ==~ /(?i)(PR-master)/) }
            }
            steps {
                container("maven") {
                    script {
                        maven.Package()
                        containerLog(name: 'maven', returnLog: false,  limitBytes: 1048576)
                    }
                }
            }
        }
        stage("Deploy to Nexus") {
            // when branch is not feature
            when {
                not {
                    branch "feature/*"
                }
                expression { return !(env.branch ==~ /(?i)(PR-master)/) }
            }
            steps {
                container('maven') {
                    script {
                        maven.Deploy(mavenArtifacts)
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                maven.PostSuccess()
                slackReports.AggregateMessage(true)
            }

        }
        failure {
            script {
                slackReports.AggregateMessage(false)
            }
        }
        always {
            cleanWs()
        }
    }
}
