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
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                echo "🧪 Testing..."
                sh 'mvn test'
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
                sh 'mvn package -DskipTests'
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
                    sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                    -Dsonar.projectName=${SONAR_PROJECT_NAME}
                    """
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
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
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

                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
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
