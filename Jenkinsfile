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
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/khalidbaddi/Task-Master-Pro.git'
            }
        }
        
        stage('compile') {
            steps {
             sh 'mvn compile'
            }
        }
        
        stage('Unit Test') {
            steps {
             sh 'mvn test'
            }
        }
        
        stage('trivy fs scan') {
            steps {
             sh 'trivy fs --format table -o fs.html .'
            }
        }
        
        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=taskmaster -Dsonar.projectKey=taskmaster \
                    -Dsonar.java.binaries=target '''
                }
            }
        }
        
        stage('Application Build') {
            steps {
             sh 'mvn package'
                
            }
        }
        
        stage('Publish Artifact') {
            steps {
             withMaven(globalMavenSettingsConfig: 'settings-maven', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }
        
        stage('Docker Build and tag') {
            steps {
                script {
             withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh 'docker build -t khaliddd/taskmaster:latest .'
                }
            }
        }}
        
        stage('trivy image scan') {
            steps {
             sh 'trivy image --format table -o image.html khaliddd/taskmaster:latest'
            }
        }
        
        stage('Docker push') {
            steps {
                script {
             withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh 'docker push khaliddd/taskmaster:latest'
                }
            }
        }}
        
        stage('k8s deploy') {
            steps {
             withKubeConfig(caCertificate: '', clusterName: 'eksdemo1', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://2B0E446B7C1012E969F1D3F9EE139FE9.gr7.ap-south-1.eks.amazonaws.com') {
                    sh 'kubectl apply -f deployment-service.yml'
                    sleep 30
                }
            }
        }
        
        stage('k8s verify') {
            steps {
             withKubeConfig(caCertificate: '', clusterName: 'eksdemo1', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://2B0E446B7C1012E969F1D3F9EE139FE9.gr7.ap-south-1.eks.amazonaws.com') {
                    sh 'kubectl get pods -n webapps'
                    sh 'kubectl get svc -n webapps'
                }
            }
        }
    }
}