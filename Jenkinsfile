pipeline{
    agent any
    tools{
        jdk 'JDK 17'
        nodejs 'node.js 16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/venkeyboda07/Zomato-Clone-DevSecOps-Jenkins.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }

        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        




stage("Docker Build & Push"){
           steps{
               script{
                  withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                      sh "docker build -t zomato ."
                      sh "docker tag zomato venkeyboda/zomato:1.0"
                      sh "docker push venkeyboda/zomato:1.0"
                   }
               }
           }
       }
       stage("TRIVY"){
           steps{
               sh "trivy image  venkeyboda/zomato:1.0 > trivy .txt"
           }
       }
    }
}