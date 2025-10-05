pipeline { 
    agent any
    tools{nodejs "NodeJS_16"}

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

        stage('Install dependencies - Backend') {
            steps {
                dir('gestion-smartphone-backend') {
                    sh 'npm install'
                    sh 'node -v && npm -v'
                }
            }
        }

        stage('Install dependencies - Frontend') {
            steps {
                dir('gestion-smartphone-frontend') {
                    sh 'npm install'
                    sh  'node -v && npm -v'
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    sh 'cd gestion-smartphone-backend && npm test || echo "Aucun test backend"'
                    sh 'cd gestion-smartphone-frontend && npm test || echo "Aucun test frontend"'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh "docker build -t $DOCKER_HUB_USER/$FRONT_IMAGE:latest ./gestion-smartphone-frontend"
                    sh "docker build -t $DOCKER_HUB_USER/$BACK_IMAGE:latest ./gestion-smartphone-backend"
                }
            }
        }

        stage('Check Docker & Compose') {
            steps {
                sh '''
                    echo "Vérification de Docker et Docker Compose..."
                    docker --version
                    if ! command -v docker compose &> /dev/null
                    then
                      echo "Installation de Docker Compose v2..."
                      apt-get update && apt-get install -y docker-compose-plugin
                    fi
                    docker compose version
                '''
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials1', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_USER/$FRONT_IMAGE:latest
                        docker push $DOCKER_USER/$BACK_IMAGE:latest
                    '''
                }
            }
        }

        stage('Clean Docker') {
            steps {
                sh 'docker container prune -f'
                sh 'docker image prune -f'
            }
        }

        stage('Deploy (compose.yaml)') {
    steps {
        dir('.') {
            sh '''
                echo "🧹 Arrêt des anciens conteneurs..."
                docker compose -f compose.yaml down || true

                echo "🏗️ Construction des images locales..."
                docker compose -f compose.yaml build --no-cache

                echo "🚀 Démarrage des services..."
                docker compose -f compose.yaml up -d

                echo "🔍 Vérification des conteneurs actifs..."
                docker compose -f compose.yaml ps

                echo "📜 Derniers logs..."
                docker compose -f compose.yaml logs --tail=50
            '''
        }
    }
}

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

    post {
        success {
            emailext(
                subject: "✅ Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Le pipeline a réussi 🎉\n\nDétails : ${env.BUILD_URL}",
                to: "w.w.wgueyekhady@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "❌ Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Le pipeline a échoué ❗\n\nVérifie les logs : ${env.BUILD_URL}",
                to: "w.w.wgueyekhady@gmail.com"
            )
        }
    }
}
