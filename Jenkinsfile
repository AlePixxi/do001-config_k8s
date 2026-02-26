pipeline {
    agent any

    parameters {
        string(
            name            : 'SERVICE NAME',
            description     : 'Enter the name of the service to deploy',
        )
        string(
            name            : 'IMAGE TAG',
            description     : 'Enter the image tag to deploy',
        )
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                credentialsId: 'credentialsgithu',
                url: 'git@github.com:AlePixxi/do001-config_k8s.git'
            }
        }

        stage('Update image tag') {
            steps {
                sh "kustomize edit set image list_products=${params.SERVICE_NAME}:${params.IMAGE_TAG}"
            }
        }

        stage('Commit & Push') {
            steps {
                sh """
                    git config user.email "jenkins@do001.com"
                    git config user.name "jenkins"
                    git add .
                    git commit -m "Promote microservice ${params.IMAGE_TAG}"
                    git push
                """
            }
        }

        stage ('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -k services/${params.SERVICE_NAME}/base"
            }
        }
    }
}