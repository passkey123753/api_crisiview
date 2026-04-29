pipeline {
    agent any
    stages {
        stage('Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/passkey123753/api_crisiview.git'
            }
        }
        stage('Install') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test || true'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                sh '''
                    sonar-scanner \
                        -Dsonar.projectKey=api_crisiview \
                        -Dsonar.sources=. \
                        -Dsonar.exclusions=node_modules/**,__tests__/**,coverage/** \
                        -Dsonar.tests=__tests__ \
                        -Dsonar.host.url=http://crisis_sonarqube:9000 \
                        -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }
        stage('Build Image') {
            steps {
                sh 'docker build -t crisis_backend .'
            }
        }
        stage('Deliver') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag crisis_backend $DOCKER_USER/crisis_backend:latest
                        docker push $DOCKER_USER/crisis_backend:latest
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                    docker stop crisis_backend || true
                    docker rm crisis_backend || true
                    docker run -d --name crisis_backend --network crisisview_default \
                        -e DB_HOST=crisis_db \
                        -e DB_USER=root \
                        -e DB_PASSWORD=root \
                        -e DB_NAME=crisiview \
                        -e DB_PORT=3306 \
                        -p 3001:3001 crisis_backend
                '''
            }
        }
    }
}
