pipeline {
    
    agent {
	    docker {
		    image 'rajatkumar216/docker-git:v1'
		    args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
	    }
    } 
    
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
        stage('Update K8S manifest & push to Repo') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
    		    sh '''
                	set -e  # Exit on error

               		git config --global user.email "kumar.rishu892@gmail.com"
                	git config --global user.name "$GIT_USERNAME"
		 	git config --global --add safe.directory /var/lib/jenkins/workspace/to-do-app
                	git remote set-url origin https://$GIT_USERNAME:$GIT_PASSWORD@github.com/$GIT_USERNAME/cicd-demo-manifests-repository.git
                	git checkout master
                	git pull --rebase --strategy=recursive -X theirs origin master

                	echo "📜 Before updating deploy.yaml:"
                	cd deploy
                	current_version=$(grep image deploy.yaml | cut -d':' -f3)
                	echo "🔢 Current image version: ${current_version}"
                	sed -i "s/${current_version}/${BUILD_NUMBER}/g" deploy.yaml

                	echo "📜 After updating deploy.yaml:"
                	cat deploy.yaml
                	git add deploy.yaml
                	git commit -m '✅ Updated the deploy.yaml via Jenkins Pipeline'

                	git push origin HEAD:master
            	     ''' 
                }
            }
        }
    }
}

