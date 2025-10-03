pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'ton_username_dockerhub'   // Mets ton vrai username Docker Hub
        FRONT_IMAGE     = 'gestion-smartphones-frontend'
        BACK_IMAGE      = 'gestion-smartphones-backend'
    }

    stages {
        stage('Checkout') {
            steps {
                // Récupérer ton code depuis GitHub
                git branch: 'main', url: 'https://github.com/khadjia3151/gestion-smartphones.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                echo '🔨 Construction des images Docker...'
                sh 'docker compose build'
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials',
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        
        stage('Deploy (optionnel)') {
            steps {
                echo '📦 Déploiement terminé (à compléter selon ton besoin).'
            }
        }
    }
}
