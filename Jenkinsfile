def label = "${env.JOB_NAME.split('/').join('.')}.${env.BUILD_NUMBER}"

podTemplate(label: label, containers: [
    containerTemplate(name: 'jnlp', image: 'jenkins/jnlp-slave:3.27-1-alpine'),
    containerTemplate(name: 'maven', image: 'maven:3.5.4-jdk-8-alpine', ttyEnabled: true, command: 'cat')
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
            kubernetes.image().withName("app").build().fromPath(".")
        }
    }
}