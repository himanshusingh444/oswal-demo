pipeline {
    agent any
    environment { GITHUB_TOKEN = credentials('github-pat') }
    stages {
        stage('Checkout') { steps { checkout scm } }
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
                            sh 'docker build -t ghcr.io/<your-username>/go-app:latest .'
                            sh 'trivy image --exit-code 1 --no-progress --severity HIGH,CRITICAL ghcr.io/<your-username>/go-app:latest'
                        }
                    }
                }
                stage('.NET') {
                    steps {
                        dir('dotnet-app') {
                            sh 'docker build -t ghcr.io/<your-username>/dotnet-app:latest .'
                            sh 'trivy image --exit-code 1 --no-progress --severity HIGH,CRITICAL ghcr.io/<your-username>/dotnet-app:latest'
                        }
                    }
                }
            }
        }
        stage('Push to GHCR') {
            steps {
                sh 'echo $GITHUB_TOKEN_PSW | docker login ghcr.io -u <your-username> --password-stdin'
                sh 'docker push ghcr.io/<your-username>/go-app:latest'
                sh 'docker push ghcr.io/<your-username>/dotnet-app:latest'
            }
        }
        stage('Create Release') {
            steps { sh 'gh release create v1.0 --notes "Initial release" --target main' }  // Install gh CLI if needed
        }
    }
}