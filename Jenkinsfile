pipeline {
    agent any
    tools{
        jdk 'jdk21'
        maven 'maven3'
    }
    environment{
        SCANNER_HOME= tool 'sonar'
    }

    stages {
        stage('git-checkout') {
            steps {
                git 'https://github.com/PrateekP97/secretsanta-generator.git'
            }
        }

        stage('Code-Compile') {
            steps {
               sh "mvn clean compile"
            }
        }
        
        stage('Unit Tests') {
            steps {
               sh "mvn test"
            }
        }
        
		stage('OWASP Dependency Check') {
            steps {
               dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }


        stage('Sonar Analysis') {
            steps {
               withSonarQubeEnv('sonar'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Santa \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Santa '''
               }
            }
        }

		 
        stage('Code-Build') {
            steps {
               sh "mvn clean package"
            }
        }

         stage('Docker Build') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred') {
                    sh "docker build -t prateek097/santa123:latest . "
                 }
               }
            }
        }

        stage('Docker Push') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred') {
                    sh "docker tag santa123 prateek097/santa123:latest"
                    sh "docker push prateek097/santa123:latest"
                 }
               }
            }
        }
		
		stage('Deploy Application') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred') {
                    sh "docker run -d -p 8081:8080 prateek097/santa123:latest . "
                 }
               }
            }
        }
	}
    
} 