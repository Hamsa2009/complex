node {
  try {
    stage('Checkout') {
		  checkout scm
    }
    stage('Init') {
		  echo 'Initializing..'
		  echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
			sh 'git --version'
		  echo "Branch: ${env.BRANCH_NAME}"
			sh 'docker -v'
		  echo 'Initialise Dockerhub login'
		  withCredentials([usernamePassword(credentialsId: 'DockerHub', passwordVariable: 'DOCKER_PWD', usernameVariable: 'DOCKER_ID')]){
		  	sh '''                            
                            echo "${DOCKER_PWD} | docker login -u ${DOCKER_ID} --password-stdin"			    
                         '''
		  }
		  	sh 'printenv'
    }
    
    stage('Build'){
		 echo 'Building Docker for Client'
		 
		 if (env.BRANCH_NAME == 'feature') {		 
		 echo 'Building Docker in Feature branch'
			sh 'docker build -t hamsa20/client-test -f ./client/Dockerfile.dev ./client'
		 }
		 else{
		 echo 'Building Docker in Master Branch'
			sh 'docker build -t hamsa20/client-test -f ./client/Dockerfile ./client'
		 }
    }
	
    boolean clientTestPassed = true
    stage('Test'){
	    echo 'Docker test for client'
		try{
			if (env.BRANCH_NAME == 'feature') {
			echo 'Test Docker in Feature Branch'
			  sh 'docker run -e CI=true hamsa20/client-test npm run test'
			}
			else{
			echo 'Test Docker in Master Branch'
				echo "We are in ${env.BRANCH_NAME} branch. Skip Docker test stage"
			}
		}
		catch (Exception e){
		    echo 'Docker test for client failed'
			clientTestPassed = false
		}
    }
	
	stage('Build Docker Images'){
		if(clientTestPassed){
		    echo 'Docker test for Client passed.'
			echo 'Building Docker image for client'
				sh 'docker build -t hamsa20/multi-client ./client'
			echo 'Building Docker image for nginx..'
				sh 'docker build -t hamsa20/multi-nginx ./nginx'
			echo 'Building Docker image for server..'
				sh 'docker build -t hamsa20/multi-server ./server'
			echo 'Building Docker image for worker..'
				sh 'docker build -t hamsa20/multi-worker ./worker'
		}
	}
	
	stage('Publish'){
		echo 'Publishing image to DockerHub..'
		echo 'Publishing image for client..'
			sh 'docker push hamsa20/multi-client'
		echo 'Publishing image for nginx..'
			sh 'docker push hamsa20/multi-nginx'
		echo 'Publishing image for server'
			sh 'docker push hamsa20/multi-server'
		echo 'Publishing image for worker..'
			sh 'docker push hamsa20/multi-worker'
	}

    stage('Cleanup'){
		echo 'Removing unused docker containers and images..'
			sh 'docker system prune -f'       		 	
    }    
  }
  catch (err) {
    throw err
  }
}
