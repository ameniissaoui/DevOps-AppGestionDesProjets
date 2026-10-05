// Atelier 4 - Travail à faire : Build + Push des images Docker et déploiement via Docker Compose
// Prérequis Jenkins :
//   - Outils "JAVA_HOME" (JDK) et "M2_HOME" (Maven) configurés (Atelier 3)
//   - Credential "dockerhub-creds" (Username with password : login Docker Hub + access token)
//   - Utilisateur jenkins membre du groupe docker (install_devops_tools.sh)
pipeline {
    agent any

    tools {
        jdk 'JAVA_HOME'
        maven 'M2_HOME'
    }

    environment {
        DOCKERHUB_USER = credentials('dockerhub-creds') // expose aussi DOCKERHUB_USER_USR / DOCKERHUB_USER_PSW
        TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'mvn -B clean package -DskipTests'
                }
            }
        }

        stage('Tests Backend') {
            steps {
                dir('backend') {
                    sh 'mvn -B test'
                }
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'backend/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Build Images Docker') {
            steps {
                sh '''
                    export DOCKERHUB_USER=$DOCKERHUB_USER_USR
                    docker compose build
                '''
            }
        }

        stage('Push Docker Hub') {
            steps {
                sh '''
                    echo "$DOCKERHUB_USER_PSW" | docker login -u "$DOCKERHUB_USER_USR" --password-stdin
                    for svc in backend frontend; do
                        img="$DOCKERHUB_USER_USR/gestion-projets-$svc"
                        docker push "$img:$TAG"
                        docker tag "$img:$TAG" "$img:latest"
                        docker push "$img:latest"
                    done
                '''
            }
        }

        stage('Deploy (Docker Compose)') {
            steps {
                sh '''
                    export DOCKERHUB_USER=$DOCKERHUB_USER_USR
                    docker compose up -d
                    docker compose ps
                '''
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'backend/target/*.jar', fingerprint: true
        }
        always {
            sh 'docker logout || true'
        }
    }
}
