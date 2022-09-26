pipeline {
    environment {
    registry = "opencartjenkins"
    registryCredential = 'acr'
    dockerImage = ''
	registryUrl = 'jenkinscicd.azurecr.io'
  }
   agent none
    stages { 
    stage('Setup Parameters') {
            steps {
                script {
                 properties([parameters([gitParameter(branch: '', branchFilter: 'origin/(.*)', defaultValue: 'dev', name: 'Branch', quickFilterEnabled: true, selectedValue: 'NONE', sortMode: 'ASCENDING_SMART', tagFilter: '*', type: 'PT_BRANCH')])])
                }
            }
    }
	stage('SCM Checkout') {
	    agent {label 'wsl'}
            steps {
                git branch: "${params.Branch}", credentialsId: 'git_credentials', url: 'https://github.com/nikhilsharma6311/opencartdocker.git'
                }
        }
        stage('Building Docker Image') {
          agent {label 'wsl'}
        steps {
            script {
                dockerImage = docker.build registry + ":$BUILD_NUMBER"
                echo "Builded Successfully"
                  }
            }
		}
    stage('SonarQube Analysis') {
		agent {label 'windows'}
		    	environment {
                scannerHome = tool 'SonarQubeScanner'
            }
        steps {
            withSonarQubeEnv('Sonar_Server') {
      		bat "${scannerHome}\\bin\\sonar-scanner \
                        -Dsonar.projectKey=OpenCart \
                        -Dsonar.host.url=http://localhost:9000 "
					}
			timeout(time: 10, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true
				}
			}
	}
    stage('Pushing Docker-Image to Dockerhub') {
	  agent {label 'wsl'}   
        steps {
             echo "Pushed Succesfully" 
             script {
                 docker.withRegistry( "http://${registryUrl}", registryCredential ) {
				         dockerImage.push("$BUILD_NUMBER")
                         dockerImage.push('latest')
                 }
            }   
        }
    }
	}
}
