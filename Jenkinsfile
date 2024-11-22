pipeline {
    agent any
    tools {
        maven 'maven3'
    }
    environment {
        SONARQUBE_HOME = tool 'sonar-scanner' 
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SyedAzherAli/Multi-Tier-BankApp-CI.git'
            }
        }
        stage('Build Application') {
            steps {
                sh "mvn clean install -DskipTests"
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }
        stage('SonarQube') {
            steps {
               withSonarQubeEnv('sonar') {
                    sh '''
                    $SONARQUBE_HOME/bin/sonar-scanner -Dsonar.projectKey=bankapp -Dsonar.projectName=bankapp -Dsonar.java.binaries=target
                    '''
                }
            }
        }
        stage('Build & Push to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-config', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests"
                }
            }
        }
        stage('Build & Tag docker image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dkrid') {
                    sh "docker build -t syedazherali/bankapp:latest ."
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image.html syedazherali/bankapp:latest"
            }
        }
        stage('Push Image to docker hub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dkrid') {
                    sh "docker push syedazherali/bankapp:latest"
                    }
                }
            }
        }
        stage("Deploy To K8's") {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'my_first_cluster', contextName: '', credentialsId: 'kube-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://D5A922D9D397FCA0B59738C13903A20D.gr7.ap-south-1.eks.amazonaws.com') {
                    sh '''
                    /var/lib/jenkins/bin/kubectl apply -f database-ds.yaml -n webapps 
                    sleep 30
                    /var/lib/jenkins/bin/kubectl apply -f application-ds.yaml -n webapps
                    '''
                }
            }
        }
        stage("Verify Deployment") {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'my_first_cluster', contextName: '', credentialsId: 'kube-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://D5A922D9D397FCA0B59738C13903A20D.gr7.ap-south-1.eks.amazonaws.com') {
                    sh '''
                    /var/lib/jenkins/bin/kubectl get svc -n webapps
                    '''
                }
            }
        }
    }
}

