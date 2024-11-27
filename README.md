


pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'nodejs16'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/prathamnandgirwar/Instagram-clone.git'
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
                withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'p', usernameVariable: 'u')]) {
    // some block
                    sh "docker login -u ${env.u} -p ${env.p}"
                       sh "docker build -t instagram ."
                       sh "docker tag instagram prathamnandgiwar/instagram:latest "
                       sh "docker push prathamnandgiwar/instagram:latest "
                    
                }
                 
                }
            
        }
        stage("TRIVY"){
            steps{
                sh "trivy image prathamnandgiwar/instagram:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name instagram -p 3000:3001 prathamnandgiwar/instagram:latest'
            }
        }
        
    }
}
