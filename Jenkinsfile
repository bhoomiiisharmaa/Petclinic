pipeline {
    agent { label 'slave1' }
    
    tools {
        maven 'MAVEN'
        jdk 'JAVA'
    }
    
    environment {
        SONAR_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = 'bhoomisharma333/petclinic'
'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                   url: 'https://github.com/bhoomisharma333/Petclinic.git'
'

            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        $SONAR_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=petclinic-bhoomi
                        -Dsonar.projectName=YourProject \
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .'
                sh 'docker tag $DOCKER_IMAGE:$BUILD_NUMBER $DOCKER_IMAGE:latest'
            }
        }
        
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE:$BUILD_NUMBER'
                    sh 'docker push $DOCKER_IMAGE:latest'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'docker rm -f myapp || true'
                sh 'docker run -d --name myapp -p 8081:8080 $DOCKER_IMAGE:latest'
            }
        }
    }
    
    post {
        always {
            sh 'docker logout || true'
        }
    }
}
