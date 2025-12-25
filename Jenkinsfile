pipeline {
    agent any

    tools {
        maven 'MAVEN'
        jdk 'JAVA'
    }

    environment {
        SONAR_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = 'bhoomisharma333/project_petclinic'
    }

    stages {

        stage('Checkout App Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/bhoomiiisharmaa/Petclinic.git'
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
                    -Dsonar.projectKey=petclinic \
                    -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .
                    docker tag $DOCKER_IMAGE:$BUILD_NUMBER $DOCKER_IMAGE:latest
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_IMAGE:$BUILD_NUMBER
                    docker push $DOCKER_IMAGE:latest
                    '''
                }
            }
        }

        stage('Checkout K8s Repo') {
            steps {
                dir('k8s') {
                    git branch: 'main',
                        url: 'https://github.com/bhoomiiisharmaa/app-k8s.git'
                }
            }
        }

        stage('Update Image Tag') {
            steps {
                sh '''
                sed -i "s|image:.*|image: $DOCKER_IMAGE:$BUILD_NUMBER|" k8s/deployment.yaml
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }
    }
}
