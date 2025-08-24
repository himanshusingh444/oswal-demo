pipeline {
    agent any
    environment {
        GITHUB_USERNAME = credentials('oswal-demo').username
        GITHUB_TOKEN_PSW = credentials('oswal-demo').password
    }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build and Test Go') {
            steps {
                dir('go-app') {
                    sh 'go mod tidy'
                    sh 'go build .'
                    sh 'go test .'
                }
            }
        }
        stage('Build and Test .NET') {
            steps {
                dir('dotnet-app') {
                    sh 'dotnet restore'
                    sh 'dotnet build'
                    sh 'dotnet test ../dotnet-app.Tests'
                }
            }
        }
        stage('Containerize and Scan') {
            parallel {
                stage('Go') {
                    steps {
                        dir('go-app') {
                            sh 'docker build -t ghcr.io/himanshusingh444/go-app:latest .'
                            sh 'trivy image --exit-code 0 --no-progress --severity HIGH,CRITICAL ghcr.io/himanshusingh444/go-app:latest'
                        }
                    }
                }
                stage('.NET') {
                    steps {
                        dir('dotnet-app') {
                            sh 'docker build -t ghcr.io/himanshusingh444/dotnet-app:latest .'
                            sh 'trivy image --exit-code 0 --no-progress --severity HIGH,CRITICAL ghcr.io/himanshusingh444/dotnet-app:latest'
                        }
                    }
                }
            }
        }
        stage('Push to GHCR') {
            steps {
                sh 'echo $GITHUB_TOKEN_PSW | docker login ghcr.io -u $GITHUB_USERNAME --password-stdin'
                sh 'docker push ghcr.io/himanshusingh444/go-app:latest'
                sh 'docker push ghcr.io/himanshusingh444/dotnet-app:latest'
            }
        }
        stage('Trigger Deployment') {
            steps {
                build job: 'deployment-pipeline', parameters: [
                    string(name: 'ENVIRONMENT', value: 'dev'),
                    string(name: 'APP_TO_DEPLOY', value: 'both')
                ]
            }
        }
        // stage('Create Release') {
        //   steps { sh 'gh release create v1.0 --notes "Initial release" --target main' }
        // }       
    }
    post {
        success {
            echo "CI completed successfully, triggering deployment to for both apps."
        }
        failure {
            echo "CI failed. Deployment not triggered."
        }
    }
}