pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes'
            label "${env.BUILD_NUMBER}.${env.JOB_NAME.split('/').reverse().join('.')}"
            containerTemplate {
                name 'jnlp'
                image 'jenkins/jnlp-slave:3.27-1-alpine'
            }
            containerTemplate {
                name 'maven'
                image 'maven:3.5.4-jdk-8-alpine'
                ttyEnabled true
                command 'cat'
            }
        }
    }
    stages {
        stage ('Info') {
            steps {
                container('maven') {
                    sh 'env'
                }
            }
        }
        stage ('Test') {
            steps {
                container('maven') {
                    sh 'mvn verify'
                }
            }
        }

        stage ('Build') {
            steps {
                container('maven') {
                    sh 'mvn install'
                }
            }
            post {
                success {
                    junit 'target/surefire-reports/**/*.xml' 
                }
            }
        }
    }
}