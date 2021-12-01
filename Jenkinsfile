pipeline {

    agent any

    environment { 
        CI = 'true'
    }

stages {
stage('Build') {
    steps {
        echo 'Building phase'       
	}
}

stage('Dependency Check'){
    steps {
        echo 'Initializing OWASP Dependency Check'
        dependencyCheck additionalArguments: '--format HTML --format XML', odcInstallation: 'Default'
    }
    
    post {
        always {
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
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
        
    stage('Start Integration Docker') {
		steps {
		    sh 'chmod +x ./jenkins/scripts/deploy.sh'
			sh './jenkins/scripts/deploy.sh'
			timeout(time: 1, unit: 'MINUTES') {
                input 'Is the test environment ready?'
            }		
		}
	}
    stage('Headless Browser Test') {
        agent {
            docker {
                image 'maven:3-alpine'
                args '-v /root/.m2:/root/.m2'
            }
        }
        
		steps {
				sh 'mvn -B -DskipTests clean package'
				sh 'mvn test'
		}
		
		post {
			always 
			    {
			        junit 'target/surefire-reports/*.xml'
					sh 'chmod +x ./jenkins/scripts/kill.sh'
					sh './jenkins/scripts/kill.sh'
                }
            }
	    }
    }
}