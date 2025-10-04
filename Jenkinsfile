pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'yayekhadygueye'   
        FRONT_IMAGE     = 'gestion-smartphones-frontend'
        BACK_IMAGE      = 'gestion-smartphones-backend'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/khadjia3151/gestion-smartphones.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                echo '🔨 Construction des images Docker...'
                sh """
                  docker build -t $DOCKER_HUB_USER/$FRONT_IMAGE:latest ./gestion-smartphone-frontend
                  docker build -t $DOCKER_HUB_USER/$BACK_IMAGE:latest ./gestion-smartphone-backend
                """
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials',
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo '🚀 Push des images sur DockerHub...'
                sh """
                  docker push $DOCKER_HUB_USER/$FRONT_IMAGE:latest
                  docker push $DOCKER_HUB_USER/$BACK_IMAGE:latest
                """
            }
        }

        stage('Deploy (optionnel)') {
            steps {
                echo '📦 Déploiement terminé (à compléter selon ton besoin).'
            }
        }
    }
}
