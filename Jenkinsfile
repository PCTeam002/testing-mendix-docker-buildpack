pipeline {
    agent any

    environment {
        ADMIN_PASSWORD = 'mendix123'
        DATABASE_ENDPOINT = 'postgres://mendix:mendix@dbhost:5432/mendix'
        IMAGE_NAME = 'pyndhotaraya/mendix-app'
        TAG = 'v1.0'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/PCTeam002/testing-mendix-docker-buildpack.git'
            }
        }

        stage('Compile MDA') {
            steps {
                sh '''
                python3 build.py --source ./src/app.mpk --destination ./build build-mda-dir
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build --build-arg ROOTFS_IMAGE=mendix-rootfs:app \
                             --build-arg BUILDER_ROOTFS_IMAGE=mendix-rootfs:builder \
                             -t $IMAGE_NAME:$TAG ./build
                '''
            }
        }

        stage('Push to Registry') {
            steps {
                sh '''
                docker login -u $DOCKER_USER -p $DOCKER_PASS
                docker push $IMAGE_NAME:$TAG
                '''
            }
        }

        stage('Deploy & Run') {
            steps {
                sh '''
                docker run -d --name mendix-app \
                  -p 9000:80 \
                  -e ADMIN_PASSWORD=$ADMIN_PASSWORD \
                  -e DATABASE_ENDPOINT=$DATABASE_ENDPOINT \
                  $IMAGE_NAME:$TAG
                '''
            }
        }
    }
}
