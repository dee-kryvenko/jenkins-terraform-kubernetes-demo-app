pipeline {
    agent {
        kubernetes {
            label 'jenkins-jenkins-slave'
            containerTemplate {
                name 'maven'
                image 'maven:3.5.4-jdk-8-alpine'
                ttyEnabled true
                command 'cat'
            }
        }
    }
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                ''' 
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