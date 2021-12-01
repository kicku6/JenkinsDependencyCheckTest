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
		    sh 'pwd'
		    sh 'chmod +x ./deploy.sh'
			sh './deploy.sh'

        	
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
                }
            }
	    }
	stage ('Checkout Analysis') {
	    steps {
	        git branch:'master', url: 'https://github.com/ScaleSec/vulnado.git'
	    }
	}
	stage ('Build Analysis') {
	    steps {
	        sh '/var/jenkins_home/apache-maven-3.6.3/bin/mvn --batch-mode -V -U -e clean verify -Dsurefire.useFile=false -Dmaven.test.failure.ignore'
	    }
	}
	stage ('Final Analysis') {
	    steps {
	        sh '/var/jenkins_home/apache-maven-3.6.3/bin/mvn --batch-mode -V -U -e checkstyle:checkstyle pmd:pmd pmd:cpd findbugs:findbugs'
	    }
	    
	    post {
	        always {
    	    recordIssues enabledForFailure: true, tools: [mavenConsole(), java(), javaDoc()] 
    	    recordIssues enabledForFailure: true, tool: checkStyle()
    	    recordIssues enabledForFailure: true, tool: spotBugs(pattern:'**/target/findbugsXml.xml')
    	    recordIssues enabledForFailure: true, tool: cpd(pattern: '**/target/cpd.xml')
    	    recordIssues enabledForFailure: true, tool: pmdParser(pattern: '**/target/pmd.xml')
    	   }
	    }
	    
	}
	
	stage ('Sonar Checkout') {
	    steps {
	        git branch:'master', url: 'https://github.com/OWASP/Vulnerable-Web-Application.git'
	    }
	}
	stage ('Sonar Quality check') {
	    steps {
	        script {
	            def scannerHome = tool 'SonarQube'
	                withSonarQubeEnv('SonarQube'){
	                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=OWASP -Dsonar.sources=. -Dsonar.host.url=http://192.168.168.135:9000 -Dsonar.login=51a75b02991b37b5d423c83cb4ba87650078231a"
	                }
	        }
	    }
	    post {
	        always {
	            recordIssues enabledForFailure: true, tool: sonarQube()
	        }
	    }
	}
	
	
    }
	post {
		success {
		            
					sh 'chmod +x /var/jenkins_home/workspace/3203labtestprep@2/kill.sh'
					sh '/var/jenkins_home/workspace/3203labtestprep@2/kill.sh'
				}
		}
			
}