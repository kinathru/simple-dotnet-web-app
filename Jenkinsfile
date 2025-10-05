pipeline {
    agent any
    stages {
        stage('Build') { 
            steps {
                sh 'dotnet restore' 
                sh 'dotnet build --no-restore' 
            }
        }
        stage('Test') {
            steps {
                sh 'dotnet test --no-build --no-restore --collect "XPlat Code Coverage"'
            }
            post {
                always {
                    recordCoverage(tools: [[parser: 'COBERTURA', pattern: '**/*.xml']], sourceDirectories: [[path: 'SimpleWebApi.Test/TestResults']])
                }
            }
        }
        stage('Deliver') {
            steps {
                sh 'dotnet publish SimpleWebApi --no-restore -o published'
                
                script {
                    def builtImage = docker.build("192.168.1.55/simple-dotnet-webapp:${env.BUILD_ID}")
                    builtImage.push()
                }
            }
            post {
                success {
                    archiveArtifacts 'published/*.*'
                }
            }
        }
    }
}