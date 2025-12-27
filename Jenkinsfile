pipeline {
    agent { label 'slave1' }

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

        stage('Update K8s Manifests Repo') {
            steps {
                dir('k8s') {
                    git branch: 'main',
                        url: 'https://github.com/bhoomiiisharmaa/app-k8s.git',
                        credentialsId: 'github-creds'

                    sh '''
                    sed -i "s|image:.*|image: $DOCKER_IMAGE:$BUILD_NUMBER|" deployment.yaml
                    git config user.email "jenkins@ci.com"
                    git config user.name "jenkins"
                    git commit -am "Update image to $BUILD_NUMBER"
                    git push origin main
                    '''
                }
            }
        }
    }
}
