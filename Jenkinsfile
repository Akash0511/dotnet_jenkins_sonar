pipeline {
    agent any
    environment{
        scannerHome = tool 'sonarscanner'
        registry = 'bhardwajakash/consolecoreapp'
        properties = null
        docker_port = 7100
        username = 'bhardwajakash'
		container_exist = "${bat(script:'docker ps -q -f name=ConsoleCoreApp', returnStdout: true).trim().readLines().drop(1).join("")}"
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
                withDockerRegistry([credentialsId: 'DockerHub',url:""]){
                    bat "docker push ${registry}:${BUILD_NUMBER}"
                }
            }
        }
        stage('Docker Deployment'){
            steps{
                script {
                    echo "ConsoleCoreApp container already exist with container id = ${env.container_exist}"
                    if (env.container_exist != null) {
                        echo "Deleting existing ConsoleCoreApp container"
                        bat "docker stop ConsoleCoreApp && docker rm ConsoleCoreApp"
                    }
                    echo "Docker Deployment"
                    bat "docker run --name ConsoleCoreApp -d -p 7100:80 ${registry}:${BUILD_NUMBER}"
                }
            }
        }
    }
}