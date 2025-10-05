pipeline {
    agent any
    parameters{
        choice(name:'BUILD_ENV', choices:['Dev', 'Staging', 'Production'], description : 'Specify the target build environment')
    }
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
            environment{
                DOCKER_REGISTRY = '192.168.1.55'
            }
            steps {
                 echo "Building docker image for ${params.BUILD_ENV} environment."
                 echo "Registry is ${env.DOCKER_REGISTRY}"
            
                sh 'dotnet publish SimpleWebApi --no-restore -o published'
                
                script {
                    def builtImage = docker.build("${env.DOCKER_REGISTRY}/simple-dotnet-webapp:${env.BUILD_ID}")
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