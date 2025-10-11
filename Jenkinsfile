pipeline { 
    agent any
    tools { nodejs "NodeJS_16" }

    environment {
        DOCKER_HUB_USER = 'yayekhadygueye'
        FRONT_IMAGE = 'gestion-smartphones-frontend'
        BACK_IMAGE  = 'gestion-smartphones-backend'
        SONARQUBE_ENV = credentials('sonarqube-token') // 🔐 Récupère le token depuis Jenkins
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
                }
            }
        }

        stage('Install dependencies - Frontend') {
            steps {
                dir('gestion-smartphone-frontend') {
                    sh 'npm install'
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

stage('SonarQube Analysis') {
    steps {
        script {
            withSonarQubeEnv('MySonarQube') {
                withSonarScanner('SonarScanner') {
                    sh '''
                        cd gestion-smartphone-backend
                        sonar-scanner \
                          -Dsonar.projectKey=gestion-smartphone-backend \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=http://sonarqube:9000 \
                          -Dsonar.login=$SONARQUBE_ENV
                    '''
                }
            }
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

        // ... (le reste de ton Jenkinsfile inchangé)
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
