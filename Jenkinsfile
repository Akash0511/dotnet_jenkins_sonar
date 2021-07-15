pipeline {
    agent any
    environment{
        scannerHome = tool 'sonarscanner'
        registry = 'bhardwajakash/consolecoreapp'
        properties = null
        docker_port = 7100
        username = 'bhardwajakash'
    }
    stages {
        stage ('Clean workspace') {
          steps {
            cleanWs()
          }
        }
        stage ('git') {
          steps {
            git 'https://github.com/Akash0511/dotnet_jenkins_sonar.git'
          }
        }
		stage('SonarQube Analysis') {
			steps{
				withSonarQubeEnv('sonarserver') {
				  bat "dotnet ${scannerHome}\\SonarScanner.MSBuild.dll begin /k:\"dummy\""
				  bat "dotnet build"
				  bat "dotnet test"
				  bat "dotnet ${scannerHome}\\SonarScanner.MSBuild.dll end"
				}
			}
		}
		stage('Docker image'){
            steps{
                echo "Docker Image Step"
                bat "dotnet publish -c Release"
                bat "docker build -t i_${username}_master --no-cache -f Dockerfile ."
            }
        }
        stage('Move image to docker hub'){
            steps{
                echo "Move Image to docker hub"
                bat "docker tag i_${username}_master ${registry}:${BUILD_NUMBER}"
                withDockerRegistry([credentialsId: 'dockerhub_id',url:""]){
                    bat "docker push ${registry}:${BUILD_NUMBER}"
                }
            }
        }
        stage('Docker Deployment'){
            steps{
                echo "Docker deployment"
                bat "docker run --name ConsoleCoreApp -d -p 7100:80 ${registry}:${BUILD_NUMBER}"
            }
        }
    }
}