pipeline {
	agent any
		
	stages {
		stage('Checkout SCM') {
			steps {
				git '/home/JenkinsDependencyCheckTest'
			}
		}

		stage('OWASP DependencyCheck') {
			steps {
				dependencyCheck additionalArguments: '--format HTML --format XML --suppression suppression.xml', odcInstallation: 'Default'
			}
		}
		
		stage('Unit Test') {
            		agent {
               	 		docker {
    			        		image 'composer:latest'
                    
                			}
	        		}
	        
			    steps {
				sh 'composer install'
				echo 'Testing Phase'
				sh './vendor/bin/phpunit --log-junit logs/unitreport.xml -c tests/phpunit.xml tests'
			    }
            
			    post {
				always {
					junit testResults: 'logs/unitreport.xml'
				}
			    }
       			 }
		stage('Integration Test'){
			parallel {
				stage('Deploy'){
					agent any
					steps {
						sh './jenkins/scripts/deploy.sh'
						input message: 'Finisheed using the web site? (Click "Proceed" to contiue)'
						
					}
				}
				stage ('Headless Browser Test') {
					agent {
						docker {
							image 'maven:3-alphine'
							args '-v /root/.m2:/root/.m2'
						}
					}
					steps {
						sh 'mvn -B -DskipTests clean package'
						sh 'mvn test'
					}
					post {
						always {
							sh './jenkins/scripts/kill.sh'
							junit 'target/surefire-reports/*.xml'
						}
					}
				}
			}
		}
		
	}	
	post {
		success {
			dependencyCheckPublisher pattern: 'dependency-check-report.xml'
		}
	}
}

