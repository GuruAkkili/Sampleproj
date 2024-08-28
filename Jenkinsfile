pipeline {
    agent any

    environment {
        CONTAINER_REGISTRY = 'marketbasket'
        RESOURCE_GROUP = 'marketbasket'
    }

    options {
        skipDefaultCheckout()
    }

    stages {
        stage('Git Clone') {
            when {
                anyOf {
                    branch 'master'
                    branch 'dev'
                    branch 'prod'
                }
            }
            steps {
                sh 'echo "clone"'
                checkout([$class: 'GitSCM', branches: [[name: '*/${env.BRANCH_NAME}']], userRemoteConfigs: [[url: 'https://github.com/VenkataVarada2023/falcon-ams-ui.git', credentialsId: 'git-clone']]])
            }
        }
        stage('Build') {
            when {
                anyOf {
                    branch 'master'
                    branch 'dev'
                    branch 'prod'
                }
            }
            steps {
                withCredentials([
                    string(credentialsId: 'azure-subscription-id-marketbasket', variable: 'AZURE_SUBSCRIPTION_ID'),
                    string(credentialsId: 'azure-tenant-id', variable: 'AZURE_TENANT_ID'),
                    // string(credentialsId: 'azure-client-secret', variable: 'AZURE_CLIENT_SECRET'),
                    // string(credentialsId: 'azure-client-id', variable: 'AZURE_CLIENT_ID'),
                    usernameColonPassword(credentialsId: 'acr-connect', variable: 'ACR_CREDENTIALS')
                ]) {
                    sh '''
                        ACR_CLIENT_ID=$(echo $ACR_CREDENTIALS | cut -d ':' -f 1)
                        ACR_SECRET=$(echo $ACR_CREDENTIALS | cut -d ':' -f 2)
                        // az login --service-principal -u $ACR_CLIENT_ID -p $ACR_SECRET -t $AZURE_TENANT_ID
                        // az account set -s $AZURE_SUBSCRIPTION_ID
                        // az acr login --name $CONTAINER_REGISTRY --resource-group $RESOURCE_GROUP --expose-token
                        // az acr build --image marketbasket.azurecr.io/market-basket:dev-$BUILD_NUMBER --registry $CONTAINER_REGISTRY --file Dockerfile .
                    '''
                }
            }
        }
        stage('K8S Deploy') {
            when {
                anyOf {
                    branch 'master'
                    branch 'dev'
                    branch 'prod'
                }
            }
            steps {
                script {
                    withKubeConfig(credentialsId: 'market-basket-aks', serverUrl: '') {
                        sh '''
                            // sed -i 's/BUILD/futuristic-1$BUILD_NUMBER/g' kuberneties/falcon-ams.yaml
                            kubectl apply -f kuberneties/falcon-ams.yaml -n falcon
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
