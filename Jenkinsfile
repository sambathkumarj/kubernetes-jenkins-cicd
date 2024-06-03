pipeline{

	agent any

	environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerhub')
	}

	stages {
		stage('Build') {

			steps {
				sh 'docker build -t sambathkumarj/jenkink8scicd:latest .
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
				script {
					kubernetesDeploy(configs: "pod.yaml, svc.yaml,secret.yaml ", kubeconfigId: "Kube-config-k8s")
				}
			}
		}
	}
}
