pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk17'
        }
	environment{
		SCANNER_HOME= tool 'sonar-scanner'
		}
    
    stages {
        stage('git_checkout') {
            steps {
                git branch: 'main', credentialsId: 'github_key', url: 'https://github.com/mydeen1206/cicd-automation.git'
            }
        }
        stage('Compile') {
            steps {
             sh 'mvn compile'
            }
        }
        stage('Test') {
            steps {
             sh 'mvn test'
			}
		}
        stage('File System Scan') {
            steps {
             sh 'trivy fs --format table -o trivy-fs-report.html .'
			}
		}
        stage('SonarQube Analysis') {
            steps {
             withSonarQubeEnv('sonar') {
				sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey \
						-Dsonar.java.binaries=. '''
					}
			}
		}
        stage('Quality_Gate') {
            steps {
				script{
				waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
				}
             
			}
		}
        stage('Build') {
            steps {
				sh 'mvn package'
             
			}
		}
        stage('Publish to Nexus') {
            steps {
				 withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
							sh 'mvn deploy'
								}
             
			}
		}
		stage('Build and Tag Docker') {
             steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
                    sh 'docker build -t mydeendevops369/automation:latest .'
							}
						}
					}
				}
		stage('File Docker Scan') {
            steps {
             sh 'trivy image --format table -o trivy-fs-report.html mydeendevops369/automation:latest'
			}
		}
		stage('Push') {
             steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
                    sh 'docker push mydeendevops369/automation:latest '
							}
						}
					}
				}
    }
}


