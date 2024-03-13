pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
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
                git branch: 'main', url: 'https://github.com/sitchatt/sample-nodejs.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Havells \
                    -Dsonar.projectKey=Havells '''
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
                       sh "docker build -t sitabjadocker/my-node-app:v1.0 ."
                       sh "docker push sitabjadocker/my-node-app:v1.0 "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image sitabjadocker/my-node-app:v1.0 > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name my-node-app -p 3000:3000 sitabjadocker/my-node-app:v1.0'
            }
        }
    }

    post {
        always {
            emailext attachLog: true,
                     attachmentsPattern: 'trivyfs.txt,trivyimage.txt', // Change this pattern to match your attachment(s)
                     body: "Project: ${env.JOB_NAME}<br/>" +
                            "Build Number: ${env.BUILD_NUMBER}<br/>" +
                            "URL: ${env.BUILD_URL}<br/>",
                     subject: "Build Status: ${currentBuild.currentResult}",
                     to: 'sitabjachatterjee158@gmail.com'
        }
    }

}
