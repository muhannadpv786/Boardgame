pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('GitCheckout') {
            steps {
               git branch: 'main', url: 'https://github.com/muhannadpv786/Boardgame.git' 
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        
    stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectkey=BoardGame \
                            -Dsonar.java.binaries=. '''
                }
            }
        }
        
       stage('Quality Gate') {
    steps {
        script {
            def qg = waitForQualityGate()
            if (qg.status != 'OK') {
                error "Quality gate failed: ${qg.status}"
            }
        }
    }
}
        
        stage('Build') {
            steps {
             sh "mvn package"   
            }
        }
        
        stage('Publish To Nexus') {
            steps {
              withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
            }
                
            
        }
     }    
        
        stage('Build and Tag Docker Image') {
            steps {
              script {
                  withDockerRegistry(credentialsId: 'docker-cred') {
                      sh "docker build -t muhannadpv/boardgame:latest ."
               }
             }
            }
            
            
        }
        
         stage('Docker Image Scan') {
            steps {
            sh "trivy image --format table -o trivy-fs-report.html  muhannadpv/boardgame:latest"    
                
            }
        }
        
        stage('Push The  Docker Image') {
            steps {
              script {
                  withDockerRegistry(credentialsId: 'docker-cred') {
                      sh "docker push  muhannadpv/boardgame:latest"
               }
            }
        }
            
            
    }
        
        stage('Deploy To Kubernates') {
            steps {
              withKubeConfig(caCertificate: '', clusterName: 'muhannad-eks.ap-south-1.eksctl.io', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://C0EF110D6C7E4291DE2F38F06309B563.gr7.ap-south-1.eks.amazonaws.com') {
                        sh "kubectl apply -f deployment-service.yaml"
}
            }
        }
        
        
         stage('Verify the Deployment') {
            steps {
             withKubeConfig(caCertificate: '', clusterName: 'muhannad-eks.ap-south-1.eksctl.io', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://C0EF110D6C7E4291DE2F38F06309B563.gr7.ap-south-1.eks.amazonaws.com') {
                   sh "kubectl get pods -n webapps"
                   sh "kubectl get svc -n webapps"
}   
            }
        }
        
        
    } 
  


   }
