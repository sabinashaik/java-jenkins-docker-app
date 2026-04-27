pipeline {

    agent any

    stages {

        stage('Clone') {

            steps {

                git 'https://github.com/sabinashaik/java-jenkins-docker-app.git'

            }

        }

        stage('Build JAR') {

            steps {

                sh 'mvn clean package -DskipTests'

            }

        }

        stage('Build Docker Image') {

            steps {

                sh 'docker build -t java-app .'

            }

        }

        stage('Run Container') {

            steps {

                sh 'docker run -d -p 8081:8080 java-app || true'

            }

        }

    }

}
 
