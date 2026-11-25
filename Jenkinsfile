pipeline {
    agent any
    tools {
        jdk 'JDK21'
        maven 'M2_HOME'
    }
    environment {
        BACKEND_IMAGE  = 'akremjouini-4ssa16-gestionprojets-backend'
        FRONTEND_IMAGE = 'akremjouini-4ssa16-gestionprojets-frontend'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-token',
                    url: 'https://github.com/akremjouini/4SSA16-AkremJouini.git'
            }
        }
        stage('Maven Clean & Compile') {
            steps { dir('backend') { sh 'mvn clean compile' } }
        }
        stage('Tests unitaires') {
            steps { dir('backend') { sh 'mvn test' } }
        }
        stage('SonarQube') {
            steps {
                dir('backend') {
                    withSonarQubeEnv('SonarQube') {
                        sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.11.0.3922:sonar -Dsonar.projectKey=4SSA16-AkremJouini -Dsonar.projectName=4SSA16-AkremJouini'
                    }
                }
            }
        }
        stage('Maven Package') {
            steps {
                dir('backend') { sh 'mvn package -DskipTests' }
                archiveArtifacts artifacts: 'backend/target/*.jar', fingerprint: true
            }
        }
        stage('Build Frontend') {
            steps { dir('frontend') { sh 'npm ci && npm run build' } }
        }
        stage('Docker Build') {
            steps {
                sh 'docker build -t ${BACKEND_IMAGE}:latest backend'
                sh 'docker build -t ${FRONTEND_IMAGE}:latest frontend'
            }
        }
        stage('Deploy (docker compose)') {
            steps { sh 'docker compose up -d --force-recreate' }
        }
    }
}
