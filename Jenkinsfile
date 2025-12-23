pipeline {
    agent any

    tools {
        jdk 'JAVA'
        maven 'MAVEN'
    }

    environment {
        IMAGE_NAME = "bhoomisharma333/petclinic"
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=Petclinic \
                    -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=target
                    '''
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format HTML', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withDockerRegistry(
                    credentialsId: 'dockerhub-creds',
                    url: 'https://index.docker.io/v1/'
                ) {
                    sh '''
                    docker build -t $IMAGE_NAME:latest .
                    docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image $IMAGE_NAME:latest'
            }
        }

        stage('Deploy To Tomcat') {
            steps {
                sh '''
                cp target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/
                '''
            }
        }
    }
}
