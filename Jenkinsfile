pipeline{
    agent any

    triggers {
        cron('H/30 * * * *')  // Runs the pipeline every 30 minutes
    }
    
    tools{
        jdk 'JDK 17'  // Ensure this matches the JDK name in Jenkins Global Tool Configuration
        nodejs 'Node.js 16'  // Ensure Node.js is installed in Jenkins
    }
    environment {
        NVD_API_KEY = credentials('NVD_API_KEY')  // Fetch API key securely
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
                 withSonarQubeEnv('sonarserver') {  // Replace 'sonar-server' with the actual name from Jenkins config
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=zomato \
                        -Dsonar.projectKey=zomato \
                        -Dsonar.host.url=http://3.111.102.112:9000 \
                        -Dsonar.login=ssqu_966c6423ee1c24cb3c95fe7a445d3406c2f97494
                    '''
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
        withCredentials([string(credentialsId: 'NVD_API_KEY', variable: 'NVD_API_KEY')]) {
            dependencyCheck additionalArguments: "--scan ./ --disableYarnAudit --disableNodeAudit --nvdApiKey=${NVD_API_KEY}", odcInstallation: 'DP-Check'
            dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        }
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