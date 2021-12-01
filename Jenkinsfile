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
		
		
	}	
	post {
		success {
			dependencyCheckPublisher pattern: 'dependency-check-report.xml'
		}
	}
}

