pipeline {
    agent any

    environment {
        MAVEN_HOME = tool 'Maven'
        ARGOCD_SERVER = 'http://argocd-server'
        ARGOCD_APP_NAME = 'java-app'
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo/java-app.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh '${MAVEN_HOME}/bin/mvn clean package'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh '${MAVEN_HOME}/bin/mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '${MAVEN_HOME}/bin/mvn sonar:sonar'
                }
            }
        }

        stage('Package Application') {
            steps {
                sh 'mv target/*.jar app.jar'
                archiveArtifacts artifacts: 'app.jar', fingerprint: true
            }
        }

        stage('Deploy to Kubernetes with Helm') {
            steps {
                sh 'helm upgrade --install java-app ./helm-chart/ --values helm-chart/values.yaml'
            }
        }

        stage('Run User Acceptance Tests') {
            steps {
                sh 'mvn verify -Puat'
            }
        }

        stage('Promote to Production using Argo CD') {
            steps {
                sh 'argocd login $ARGOCD_SERVER --username admin --password $ARGOCD_PASSWORD'
                sh 'argocd app sync $ARGOCD_APP_NAME'
            }
        }
    }
}
