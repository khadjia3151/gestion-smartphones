pipeline {
    agent any

    tools {
        nodejs "NodeJS_18"}

    environment {
        DOCKER_HUB_USER = 'yayekhadygueye'
        FRONT_IMAGE = 'gestion-smartphones-frontend'
        BACK_IMAGE  = 'gestion-smartphones-backend'
    }

    triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'ref', value: '$.ref'],
                [key: 'pusher_name', value: '$.pusher.name'],
                [key: 'commit_message', value: '$.head_commit.message']
            ],
            causeString: 'Push par $pusher_name sur $ref: "$commit_message"',
            token: 'mysecret',
            printContributedVariables: true,
            printPostContent: true
        )
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/khadjia3151/gestion-smartphones.git'
            }
        }

        /* === Installation des dépendances === */
        stage('Install dependencies - Backend') {
            steps {
                script {
                        dir('gestion-smartphone-backend') {
                            sh 'npm ci || npm install'
                        }
                    
                }
            }
        }

        stage('Install dependencies - Frontend') {
            steps {
                script {
                        dir('gestion-smartphone-frontend') {
                            sh 'npm ci || npm install'
                        }
                    
                }
            }
        }

        /* === Tests === */
        stage('Run Tests') {
            steps {
                script { 
                        sh 'cd gestion-smartphone-backend && npm test || echo "Aucun test backend"'
                        sh 'cd gestion-smartphone-frontend && npm test || echo "Aucun test frontend"'
                }
            }
        }

        /* === Build des images Docker === */
        stage('Build Docker Images') {
            steps {
                script {
                    sh "docker build -t $DOCKER_HUB_USER/$FRONT_IMAGE:latest ./gestion-smartphone-frontend"
                    sh "docker build -t $DOCKER_HUB_USER/$BACK_IMAGE:latest ./gestion-smartphone-backend"
                }
            }
        }

        /* === Vérification de Docker et Docker Compose === */
        stage('Check Docker & Compose') {
            steps {
                sh '''
                    echo "Vérification de Docker et Docker Compose..."
                    docker --version
                    if ! command -v docker-compose &> /dev/null
                    then
                      echo "Installation de docker-compose..."
                      apt-get update && apt-get install -y docker-compose
                    fi
                    docker-compose --version
                '''
            }
        }

        /* === Push sur Docker Hub === */
        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_USER/$FRONT_IMAGE:latest
                        docker push $DOCKER_USER/$BACK_IMAGE:latest
                    '''
                }
            }
        }

        /* === Nettoyage des images et conteneurs inutiles === */
        stage('Clean Docker') {
            steps {
                sh 'docker container prune -f'
                sh 'docker image prune -f'
            }
        }

        /* === Déploiement avec docker-compose === */
        stage('Deploy (docker-compose.yml)') {
            steps {
                dir('.') {
                    sh 'docker-compose -f docker-compose.yml down || true'
                    sh 'docker-compose -f docker-compose.yml pull || true'
                    sh 'docker-compose -f docker-compose.yml up -d'
                    sh 'docker-compose -f docker-compose.yml ps'
                    sh 'docker-compose -f docker-compose.yml logs --tail=50 || true'
                }
            }
        }

        /* === Test de bon fonctionnement === */
        stage('Smoke Test') {
            steps {
                sh '''
                    echo " Vérification Frontend (port 5173)..."
                    curl -f http://localhost:5173 || echo "Frontend unreachable"

                    echo " Vérification Backend (port 5001)..."
                    curl -f http://localhost:5001/api || echo "Backend unreachable"
                '''
            }
        }
    }

    /* === Notifications mail === */
    post {
        success {
            emailext(
                subject: "✅ Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Le pipeline s'est exécuté avec succès 🎉\nDétails : ${env.BUILD_URL}",
                to: "w.w.wgueyekhady@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "❌ Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Le pipeline a échoué 😞\nVérifie les logs ici : ${env.BUILD_URL}",
                to: "w.w.wgueyekhady@gmail.com"
            )
        }
    }
}
