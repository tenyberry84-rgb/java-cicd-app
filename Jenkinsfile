pipeline {
    agent any
    
    environment {
        SONAR_SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = "robin23244/java-cicd-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-cred', url: 'https://github.com/tenyberry84-rgb/java-cicd-app.git'
            }
        }
        
        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                        ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=java-app \
                        -Dsonar.projectName=java-app \
                        -Dsonar.sources=src/main/java \
                        -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }
        
        stage('Docker Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-cred') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push('latest')
                    }
                }
            }
        }
        
        stage('Update K8s Manifest') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-cred', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    sh """
                        sed -i 's|robin23244/java-cicd-app:.*|robin23244/java-cicd-app:${DOCKER_TAG}|g' k8s/deployment.yaml
                        git config user.email 'jenkins@jenkins.com'
                        git config user.name 'Jenkins'
                        git add k8s/deployment.yaml
                        git commit -m 'Update image tag to ${DOCKER_TAG}'
                        git push https://${GIT_TOKEN}@github.com/tenyberry84-rgb/java-cicd-app.git main
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}