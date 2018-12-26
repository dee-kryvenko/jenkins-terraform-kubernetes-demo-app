def label = "jenkins-terraform-kubernetes-demo-app.${env.BUILD_NUMBER}.${UUID.randomUUID().toString()}"

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
        if (env.BRANCH_NAME == 'master') {
            stage ('Deploy') {
                container('awscli') {
                    env.AWS_ACCOUNT_ID = sh(script: 'aws sts get-caller-identity --output text --query "Account"', returnStdout: true).trim()
                    sh 'aws ecr get-login --region us-east-1 --no-include-email > ecr_auth.sh'
                }
                def image = "${env.AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/jenkins-terraform-kubernetes-demo-apps:${env.BUILD_NUMBER}"
                container('docker') {
                    sh 'chmod +x ./ecr_auth.sh && ./ecr_auth.sh'
                    sh "docker build -t ${image} ."
                    sh "docker push ${image}"
                }
                def ingress = sh(script: "basename '${env.JENKINS_URL}'", returnStdout: true).trim()
                container('helm') {
                    sh "helm upgrade --namespace 'jenkins' --install --force 'jenkins-terraform-kubernetes-demo-app' './helm/jenkins-terraform-kubernetes-demo-app' --set app.container.image='${image}' --set app.host='${ingress}' --reset-values --wait"
                }
            }
        }
    }
}