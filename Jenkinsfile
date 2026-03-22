pipeline {
    agent any
    tools{
        maven "maven-trial"
        jdk "jdk17"
        // dockerTool "docker"
    }
    environment {
        SCANNER_HOME = tool "sonarQube-trial"
        PATH = "/usr/local/bin:/opt/homebrew/bin:/usr/bin:/bin:${env.PATH}"
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: '8adf0106-4951-4467-a024-8a819e6d1d7d', poll: false, url: 'https://github.com/prajwalbalapure1/3-tier-application-CICD.git'            }
        }
        stage('Compile') {
            steps {
                sh "mvn clean compile -DskipTests=True"
            }
        }
        stage('dependency check OWASP') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'owasp-security-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server-personal') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Shopping-Cart \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Shopping-Cart '''
                }
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=True"
            }
        }
        stage('docker check') {
            steps {
                sh "docker version"
                sh "docker context ls"
            }
        }
        stage('docker image build and push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker build -t shopping-cart1 -f docker/Dockerfile ."
                    sh "docker tag shopping-cart1 prajwalbalapure25/shopping-cart1:latest"
                    sh "docker push prajwalbalapure25/shopping-cart1:latest"
                    }
                }
            }
        }
