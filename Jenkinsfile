pipeline { 
 agent any

tools {
jdk 'jdk17'
maven 'maven3'
}

enviornment {
SCANNER_HOME= tool 'sonar-scanner'
}

stages {
stage('Git Checkout') {
 
steps {
git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/muhannadpv786/Boardgame.git'
}
}

stage('Compile') { steps {
sh "mvn compile"
}
}

stage('Test') { steps {
sh "mvn test"
}
}

stage('File System Scan') { steps {
sh "trivy fs --format table -o trivy-fs-report.html ."
}
}

stage('SonarQube Analsyis') { steps {
withSonarQubeEnv('sonar') {
sh ''' $SCANNER_HOME/bin/sonar-scanner - Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
-Dsonar.java.binaries=. '''
}
}
}

stage('Quality Gate') { steps {
script {
waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
}
}
}

stage('Build') { steps {
sh "mvn package"
}
}

stage('Publish To Nexus') { steps {
withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
 
sh "mvn deploy"
}
}
}

stage('Build & Tag Docker Image') { steps {
script {
withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') { sh "docker build -t muhannadpv/boardshack:latest ."
}
}
}
}

stage('Docker Image Scan') { steps {
sh "trivy image --format table -o trivy-image-report.html muhannadpv/boardshack:latest "
}
}

stage('Push Docker Image') { steps {
script {
withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') { sh "docker push muhannadpv/boardshack:latest"
}
}
}
}
stage('Deploy To Kubernetes') { steps {
withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.8.146:6443') {
sh "kubectl apply -f deployment-service.yaml"
}
}
}

stage('Verify the Deployment') { steps {
withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.8.146:6443') {
sh "kubectl get pods -n webapps"
sh "kubectl get svc -n webapps"
}
}
}
 
}
}

