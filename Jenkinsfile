pipeline {
    agent any

    environment {
        IMAGE_NAME = "devopsmeeraja/python-app"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/MeerajaMohandas/New_python_app.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate

                    pip install --upgrade pip
                    pip install -r requirements.txt

                    python3 -m py_compile app.py
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                    -t $IMAGE_NAME:$BUILD_NUMBER .
                '''
            }
        }

        stage('Docker Push') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASS" | \
                        docker login \
                        -u "$DOCKER_USER" \
                        --password-stdin

                        docker push \
                        $IMAGE_NAME:$BUILD_NUMBER

                        docker logout
                    '''
                }
            }
        }

        stage('Rolling Deployment') {
            steps {

                sh '''
                    kubectl set image deployment/flask-app \
                    flask-app=$IMAGE_NAME:$BUILD_NUMBER

                    kubectl rollout status \
                    deployment/flask-app
                '''
            }
        }
    }

    post {

        success {
            echo 'Deployment successful!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}
