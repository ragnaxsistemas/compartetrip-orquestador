def project = 'r-poc-247201'
def appName = 'compartetrip-orquestador'
def imageTag = "gcr.io/${project}/${appName}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"

pipeline {
    agent {
        node {
            label('slave-java-01')
        }
    }
    stages {
        stage('Download Dependency') {
            steps {
                sh """
                /opt/apache-maven-3.6.1/bin/mvn clean package
                """
            }
        }
        stage('SonarQube Analysis') {
            steps {
                sh """
                /opt/apache-maven-3.6.1/bin/mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar \
                -Dsonar.host.url=http://34.70.106.141 -Dsonar.login=ecd2422380147418af5c9107b99040804eb5da5b -Dsonar.sources=pom.xml,src/main/,target/classes
                """
            }
        }
        stage('Construct Artifact') {
            steps {
                sh """
                /opt/apache-maven-3.6.1/bin/mvn clean install
                """
            }
        }
        stage('Build & Push Image') {
            steps {
                sh """
                PYTHONUNBUFFERED=1 /snap/bin/gcloud builds submit -t ${imageTag} .
                """
            }
        }
        stage('Run in Cloud RUN') {
            steps {
                sh """
                PYTHONUNBUFFERED=1 /snap/bin/gcloud beta run deploy ${appName}-${env.BRANCH_NAME}-${env.BUILD_NUMBER} --image ${imageTag} --platform managed --region us-central1
                """
            }
        }
    }
}