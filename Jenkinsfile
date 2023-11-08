@Library('pipeline-library-demos')_
pipeline {
    agent { label 'ci-cd-slave' }
    tools {
        jdk 'java17'
        maven 'Maven3'
    }
    environment {
	    APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            registryCredential = 'dockerhub'
	    DOCKER_USER = 'hemanth2220'
	    IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	    registry = "$DOCKER_USER/$APP_NAME"
	    JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")

  
    }

    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/hemanthrh/register-app.git'
                }
        }

        stage("Buildign Package"){
           steps {
                echo" Building in Shared Library"
           }
	}

       stage("Test Application"){
           steps {
                 sh "mvn test"
		 sh "echo welcome"
           }
       }

        stage("SonarQube Analysis"){
           steps {
	           script {
		        withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                        sh "mvn sonar:sonar"
		        }
	           }	
           }
       }
stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }	
            }

        }

	            stage('Docker Build image') {
steps{
script {
docker_image = docker.build "${IMAGE_NAME}"
	
}
}
}
	    
	    
stage('Docker Push image') {
steps{
script {
docker.withRegistry( '', registryCredential ) {
docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
}
}
}
}

       stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image hemanth2220/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }





stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       }

       stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' '192.168.163.130:8080/job/git-ops-cd/buildWithParameters?token=gitops-token'"
                }
            }
       }

       }
}

