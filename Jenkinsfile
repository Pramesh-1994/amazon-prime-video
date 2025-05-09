pipeline{
    agent any
    tools{
        jdk 'jdk'
        nodejs 'node17'
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
                git branch: 'main', url: 'https://github.com/pramesh-1994/amazon-prime-video.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('SonarQube') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=amazon-prime-video \
                    -Dsonar.projectKey=amazon-prime-video \
		    -Dsonar.login=YOUR_SONARQUBE_TOKEN '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
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
                       sh "sudo docker build -t amazon-prime-video ."
                       sh "sudo docker tag amazon-prime-video pramesh1994/amazon-prime-video:latest "
                       sh "sudo docker push pramesh1994/amazon-prime-video:latest "
                    }
                }
            }
        }
		stage('Docker Scout Image') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh 'docker-scout quickview pramesh1994/amazon-prime-video:latest'
                       sh 'docker-scout cves pramesh1994/amazon-prime-video:latest'
                       sh 'docker-scout recommendations pramesh1994/amazon-prime-video:latest'
                   }
                }
            }
        }

        stage("TRIVY-docker-images"){
            steps{
                sh "trivy image pramesh1994/amazon-prime-video:latest > trivyimage.txt" 
            }
        }
        stage('App Deploy to Docker container'){
            steps{
                sh 'docker run -d --name amazon-prime-video -p 3000:3000 pramesh1994/amazon-prime-video:latest'
            }
        }

    }
    post {
    always {
        script {
            def buildStatus = currentBuild.currentResult
            def buildUser = currentBuild.getBuildCauses('hudson.model.Cause$UserIdCause')[0]?.userId ?: 'Github User'
            
            emailext (
                subject: "Pipeline ${buildStatus}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <p>This is a Jenkins amazon-prime-video CICD pipeline status.</p>
                    <p>Project: ${env.JOB_NAME}</p>
                    <p>Build Number: ${env.BUILD_NUMBER}</p>
                    <p>Build Status: ${buildStatus}</p>
                    <p>Started by: ${buildUser}</p>
                    <p>Build URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """,
                to: 'pramesh.maurya1994@gmail.com',
                from: 'pramesh.maurya1994@gmail.com',
                replyTo: 'pramesh.maurya1994@gmail.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
            )
           }
       }

    }

}
