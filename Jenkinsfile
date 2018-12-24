def label = "${env.JOB_NAME.split('/').join('.')}.${env.BUILD_NUMBER}"

podTemplate(label: label, containers: [
    containerTemplate(name: 'jnlp', image: 'jenkins/jnlp-slave:3.27-1-alpine'),
    containerTemplate(name: 'maven', image: 'maven:3.5.4-jdk-8-alpine', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'awscli', image: 'mesosphere/aws-cli:1.14.5', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'dind', image: 'docker:18.03.1-ce-dind', privileged: true),
    containerTemplate(name: 'docker', image: 'docker:18.03.1-ce', envVars: [envVar(key: 'DOCKER_HOST', value: 'tcp://localhost:2375')], ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'helm', image: 'alpine/helm:2.11.0', ttyEnabled: true, command: 'cat')
]) {
    node(label) {
        stage('Test') {
            checkout scm
            container('maven') {
                sh 'mvn verify'
            }
            junit 'target/surefire-reports/**/*.xml'
        }
        stage('Build') {
            checkout scm
            container('maven') {
                sh 'mvn package'
            }
        }
        stage ('Image') {
            container('awscli') {
                sh 'aws ecr get-login --region us-east-1 --no-include-email > ecr_auth.sh'
            }
            container('docker') {
                sh 'chmod +x ./ecr_auth.sh && ./ecr_auth.sh'
                sh 'docker build -t 427657384269.dkr.ecr.us-east-1.amazonaws.com/jenkins-terraform-kubernetes-demo-app .'
                sh 'docker push 427657384269.dkr.ecr.us-east-1.amazonaws.com/jenkins-terraform-kubernetes-demo-app'
            }
        }
    }
}