pipeline {
	agent any
	stages {
		stage ('Build Backend') {
			steps {
				bat 'mvn clean package -DskipTests=true'
			}
		}
		
		stage ('Unit Tests') {
			steps {
				bat 'mvn test'
			}
		}
		
		stage ('Sonar Analysis') {
			environment {
				scannerHome = tool 'SONAR_SCANNER'
			}
			
			steps {
				withSonarQubeEnv('SONAR_LOCAL') {
					bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=80e0da1365a30d4a76aa8df3d8b9ed5c9f372632 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
				}
			}
		}
		
		stage ('Quality Gate') {
			steps {
				sleep(20)
				timeout(time: 1, unit: 'MINUTES') {
					waitForQualityGate abortPipeline: true
				}
			}
		}
		
		stage ('Deploy Backend') {
			steps {
				deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
			}
		}
		
		stage ('API Test') {
			steps {
				dir('api-test') {
					git url: 'https://github.com/josmar-leite/tasks-api-test'
					bat 'mvn test'
				}
			}
		}
		
		stage ('Deploy Frontend') {
			steps {
				dir('frontend') {
					git url: 'https://github.com/josmar-leite/tasks-frontend'
					bat 'mvn clean package'
					deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
				}
			}
		}
		
		stage ('Functional Test') {
			steps {
				dir('functional-test') {
					git url: 'https://github.com/josmar-leite/tasks-funcional-tests'
					bat 'mvn test'
				}
			}
		}
	}
}