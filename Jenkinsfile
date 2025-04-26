pipeline {
    
    agent any 
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
	DOCKER_IMAGE = "rajatkumar216/myfirstrepo:${BUILD_NUMBER}"
    }
    stages {
        
        stage('Checkout'){
           steps {
                git credentialsId: 'd2f8ce69-4988-403b-b65d-4970ed5fb9de', 
                url: 'https://github.com/Rajat892/cicd-end-to-end',
                branch: 'master'
           }
        }

        stage('Build Docker'){
            steps{
                script{
                    sh '''
                    echo 'Buid Docker Image'
                    docker build -t rajatkumar216/myfirstrepo:${BUILD_NUMBER} .
                    '''
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        def docker_image = docker.image("${DOCKER_IMAGE}")
                        docker_image.push()
                    }
                }
            }
        
        stage('Checkout K8S manifest SCM'){
            steps {
                git credentialsId: 'd2f8ce69-4988-403b-b65d-4970ed5fb9de', 
                url: 'https://github.com/Rajat892/cicd-end-to-end.git',
                branch: 'master'
            }
        }
        
        stage('Update K8S manifest & push to Repo'){
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: 'd2f8ce69-4988-403b-b65d-4970ed5fb9de', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh '''
                        cat deploy.yaml
                        sed -i '' "s/32/${BUILD_NUMBER}/g" deploy.yaml
                        cat deploy.yaml
                        git add deploy.yaml
                        git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                        git remote -v
                        git push https://github.com/Rajat892/cicd-demo-manifests-repo.git HEAD:main
                        '''                        
                    }
                }
            }
        }
    }
}
}
