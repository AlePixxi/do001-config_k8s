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
        
        stage('Check Current Image Tag') {
            steps {
                script {

                    // Usa kustomize build per ottenere il tag attuale
                    def currentTag = sh(
                        script: """
                        export PATH=$HOME/bin:$PATH
                        cd services/${params.SERVICE_NAME}/base
                        cat kustomization.yaml | grep 'newTag:' | sed 's/.*://'
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Tag corrente: ${currentTag}, Tag richiesto: ${params.IMAGE_TAG}"

                    // Flag per decidere se aggiornare
                    env.NEED_UPDATE = (currentTag != params.IMAGE_TAG) ? "true" : "false"
                }
            }
        }

        stage('Update image tag') {
            when {
                expression { env.NEED_UPDATE == "true" }
            }
            steps {
                sh """
                    ls -l
                    export PATH=$HOME/bin:$PATH
                    cd services/${params.SERVICE_NAME}/base
                    kustomize edit set image list_products=${params.SERVICE_NAME}:${params.IMAGE_TAG}
                """
            }
        }

        stage('Commit & Push') {
            when {
                expression { env.NEED_UPDATE == "true" }
            }
            steps {
                withCredentials([string(credentialsId: 'token-github', variable: 'TOKEN')]) {

                    sh """
                        git config user.email "jenkins@do001.com"
                        git config user.name "jenkins"
                        git add .
                        git commit -m "Promote microservice ${params.IMAGE_TAG}"
                        git push -u https://${TOKEN}@github.com/AlePixxi/do001-config_k8s.git main
                    """
                }
            }
        }

        stage ('Deploy to Kubernetes') {
            steps {
                sh """
                        export KUBECONFIG=/var/lib/jenkins/.kube/config
                        kubectl apply --validate=false -k services/${params.SERVICE_NAME}/base
                    """
            }
        }
    }
}