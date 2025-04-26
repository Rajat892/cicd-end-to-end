pipeline {
    
    agent any 
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
	DOCKER_IMAGE = "rajatkumar216/myfirstrepo:${BUILD_NUMBER}"
	GIT_USERNAME= "Rajat892"
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
	}
        stage('Checkout K8S manifest SCM'){
            steps {
                git credentialsId: 'd2f8ce69-4988-403b-b65d-4970ed5fb9de', 
                url: 'https://github.com/Rajat892/cicd-end-to-end.git',
                branch: 'master'
            }
        }
        
        stage('Update K8S manifest & push to Repo') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh '''
                        git config --global user.email "kumar.rishu892@gmail.com"
                        git config --global user.name "$GIT_USERNAME"
			cat deploy/deploy.yaml
			current_version=`cat deploy/deploy.yaml | grep image | cut -d':' -f 3`
			sed -i "s/${current_version}/${BUILD_NUMBER}/g" deploy/deploy.yaml
                        cat deploy/deploy.yaml
                        git add deploy/deploy.yaml
                        git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                        git push https://$GIT_PASSWORD@github.com/$GIT_USERNAME/cicd-demo-manifests-repo.git HEAD:master
                    '''
                }
            }
        }
    }
}

