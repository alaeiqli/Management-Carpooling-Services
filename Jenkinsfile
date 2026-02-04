pipeline {

    agent any

    tools {
        maven 'Maven'
        jdk 'Java17'
    }

    environment {
        SONAR_PROJECT_KEY = 'Management-Carpooling-Services'
        SONAR_PROJECT_NAME = 'Management-Carpooling-Services'
        DOCKER_IMAGE = 'yassiramraoui/management-carpooling-services'
        DOCKER_TAG = 'latest'
    }

    stages {

        stage('Clone') {
            steps {
                echo "📥 Cloning repository..."
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "🏗️ Build..."
                bat 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                echo "🧪 Testing..."
                bat 'mvn test'
            }

            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                echo "📦 Packaging..."
                bat 'mvn package -DskipTests'
            }

            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar, target/*.war'
                }
            }
        }

        stage('SonarQube') {
    steps {
        echo "🔍 Sonar Analysis..."

        withSonarQubeEnv('sonar_integration') {
            withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                bat """
                mvn sonar:sonar ^
                -Dsonar.projectKey=${SONAR_PROJECT_KEY} ^
                -Dsonar.projectName=${SONAR_PROJECT_NAME} ^
                -Dsonar.host.url=${SONAR_HOST_URL} ^
                -Dsonar.login=%SONAR_TOKEN%
                """
            }
        }
    }
}


        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo "🐳 Docker Build..."
                bat "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
        }

        stage('Docker Push') {
            steps {
                echo "🚀 Docker Push..."

                withCredentials([
                  usernamePassword(
                    credentialsId: 'DockerHub',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                  )
                ]) {

                    bat """
                    echo %PASS% | docker login -u %USER% --password-stdin
                    docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }
    }

    post {

        always {
            cleanWs()
        }

        success {
            echo "✅ SUCCESS"
        }

        failure {
            echo "❌ FAILED"
        }
    }
}
