pipeline{

	agent any

	environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerhub')
	}

	stages {
		stage('Build') {

			steps {
				sh 'docker build -t sambathkumarj/kube-jenkin-cicd:latest .'
			}
		}

		stage('Login') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}

		stage('Push') {

			steps {
				sh 'docker push sambathkumarj/kube-jenkin-cicd:latest'
			}
		}

		stage('Deploy to Kubernetes'){
			steps{
				withKubeConfig([credentialsId: 'Kube-config-k8s']) {
					sh 'kubectl apply -f pod.yaml'
					sh 'kubectl apply -f svc.yaml'
					sh 'kubectl apply -f secret.yaml'
				}
			}
		}	
	}
}
