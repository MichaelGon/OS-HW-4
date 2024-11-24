pipeline {
    agent any

    tools {
        maven 'mvn'
        jdk 'jdk'
        allure 'allure'
    }

    environment {
        SONAR_PROJECT_KEY = 'todo-app'
        SONAR_PROJECT_NAME = 'App'
        SONAR_HOST_URL = 'https://adc9-83-220-239-246.ngrok-free.app'
        SONAR_LOGIN = credentials('sonarqube')
    }

    stages {
        stage('Clone') {
            steps {
                sh 'git clone https://github.com/MichaelGon/OS_HW_4'
            }
        }
        stage('libs') {
            steps {
                dir('todo-app') {
                    sh 'apt-get update'
                    sh 'apt install -y maven'
                }
            }
        }

        stage('Build') {
            steps {
                dir('todo-app') {
                    sh 'mvn clean install -DskipTests'
                }
            }
        }

        stage('Testing') {
            steps {
                dir('todo-app') {
                    sh 'mvn clean test'
                }
            }
            post {
                always {
                    allure([
                        includeProperties: false,
                        jdk: '',
                        properties: [],
                        reportBuildPolicy: 'ALWAYS',
                        results: [[path: 'todo-app/allure-results']]
                    ])
                }
            }
        }

        stage('SonarQube') {
            steps {
                dir('myapp') {
                    withSonarQubeEnv('SonarQubeServer') {
                        sh '''
                            mvn sonar:sonar \
                            -Dsonar.projectKey="todo-app" \
                            -Dsonar.projectName="App" \
                            -Dsonar.host.url="https://adc9-83-220-239-246.ngrok-free.app" \
                            -Dsonar.login=credentials('sonarqube')
                        '''
                    }
                }
            }
        }
        stage('Install Docker') {
            steps {
                sh 'apt-get update'
                sh 'apt-get install -y docker.io'
                sh 'systemctl enable docker'
                sh 'dockerd > /dev/null 2>&1 &'

            }
        }
        stage('Docker Build') {
            steps {
                dir('myapp'){
                    script {
                        sh 'docker build -t mikhailgon/todo-app:latest .'
                    }
                }
            }
        }
        stage('Docker Push') {
            steps {
                dir('myapp') {
                    script {
                        withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'user', passwordVariable: 'password')]) {
                            sh 'echo $password | docker login -u $user --password-stdin'
                        }
                        sh 'docker push mikhailgon/myapp:latest'
                    }
                }
            }
        }
    }
}