pipeline {
    agent {label 'master' }
    tools{
        git "Default"
        maven "Maven"
    }
    environment{
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: 'Github_PAT', poll: false, url: 'https://github.com/vasanthvkjc/Ekart.git'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage('SonarQube Analysis') {
            steps{
                withSonarQubeEnv('sonarqube') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Ekart -Dsonar.projectName='Ekart' -Dsonar.java.binaries=./target/classes '''
                }
            }
        }
        stage('OWASP') {
            steps{
                dependencyCheck additionalArguments : '--scan ./ ', odcInstallation : 'OWASP'
                dependencyCheckPublisher pattern : '**/dependency-check-report.xml'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }
        stage('DockerBuild') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'DockerRegistry', toolName: 'Docker') {
                        sh 'docker build -t ekart:latest -f docker/Dockerfile .'
                        sh 'docker tag ekart:latest vasanthvkjc/ekart:latest'
                        sh 'docker push vasanthvkjc/ekart:latest'
                    }
                }
                
            }
        }
        stage('DockerDeploy') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'DockerRegistry', toolName: 'Docker') {
                        sh 'docker run -d --name ekart -p 8070:8070 vasanthvkjc/ekart:latest'
                    }
                }
                
            }
        }
    }
}
