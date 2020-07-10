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
                        image: maven:3.6.3-jdk-8                        
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
