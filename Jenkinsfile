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
        dependencyCheck additionalArguments: '--format HTML --format XML --out /var/jenkins_home/logs/dependency_check/${BUILD_NUMBER}', odcInstallation: 'Default'
    }
    
    post {
        always {
                sh 'cp /var/jenkins_home/logs/dependency_check/${BUILD_NUMBER}/dependency-check-report.xml ${WORKSPACE}'
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
        sh './vendor/bin/phpunit --log-junit /var/jenkins_home/logs/tests/unit/${BUILD_NUMBER}_unitreport.xml -c tests/phpunit.xml tests'
    }
    
    post {
        always {
                sh 'cp /var/jenkins_home/logs/tests/unit/${BUILD_NUMBER}_unitreport.xml ${WORKSPACE}'
                junit testResults: '*.xml'
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