pipeline {
    agent any

    parameters {
        string(
            name            : 'SERVICE_NAME',
            description     : 'Enter the name of the service to deploy',
        )
        string(
            name            : 'IMAGE_TAG',
            description     : 'Enter the image tag to deploy',
        )
    }

    stages {
        stage('Clear workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Install Kustomize') {
            steps {
                sh '''
                    mkdir -p $HOME/bin
                    curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
                    mv kustomize $HOME/bin/
                    export PATH=$HOME/bin:$PATH
                    kustomize version
                '''
            }
        }
        
        stage('Checkout') {
            steps {
                git branch: 'main',
                credentialsId: 'credentialsgithu',
                url: 'https://github.com/AlePixxi/do001-config_k8s.git'
            }
        }

        stage('Update image tag') {
            steps {
                sh """
                    export PATH=$HOME/bin:$PATH
                    kustomize edit set image list_products=${params.SERVICE_NAME}:${params.IMAGE_TAG}
                """
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
                sh """
                    export PATH=$HOME/bin:$PATH
                    kubectl apply -k services/${params.SERVICE_NAME}/base
                """
            }
        }
    }
}