pipeline{

	agent any

	environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerhub')
	}

	stages {
		stage('Build') {

			steps {
				sh 'docker build -t sambathkumarj/jenkink8scicd:latest .'
			}
		}

		stage('Login') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}

		stage('Push') {

			steps {
				sh 'docker push sambathkumarj/jenkink8scicd:latest'
			}
		}

		stage('Deploy to Kubernetes'){
			steps{
				withKubeConfig([credentialsId: 'Kube-config-k8s']) {
					sh 'microk8s kubectl apply -f pod.yaml'
					sh 'microk8s kubectl apply -f svc.yaml'
					sh 'microk8s kubectl apply -f secret.yaml'
				}
			}
		}	
	}
}
