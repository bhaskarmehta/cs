pipeline {
    agent any
    parameters {
        string(name: 'BASE_URL', defaultValue: '/user1', description: 'Base URL for Jupyter Notebook')
        string(name: 'USER', defaultValue: 'user1', description: 'User directory')
        string(name: 'QUOTA_GB', defaultValue: '5', description: 'Storage quota in GB')
        string(name: 'CPU', defaultValue: '1', description: 'CPU in Core')
        string(name: 'RAM', defaultValue: '2Gi', description: 'Memory in GB')
        string(name: 'PASSWORD', defaultValue: 'test@123', description: 'Enter your password')
    }
    environment {
        // Define environment variables for GKE configuration
        GKE_PROJECT_ID = 'winter-berm-453805-n5'
        GKE_CLUSTER_NAME = 'test-cluster'
        GKE_CLUSTER_ZONE = 'asia-south1-a'
        GKE_CLUSTER_REGION = 'asia-south1'
        //GKE_CREDENTIAL_ID = 'gke'
        QUOTA_KB = "${params.QUOTA_GB.toInteger() * 1024 * 1024}"
    }

    stages {

        stage('Validate Parameters') {
            steps {
                script {
                    // Validate if PASSWORD parameter is empty
                    if (params.PASSWORD.trim() == '') {
                        error "Password parameter is required. Please provide a valid password."
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                // Authenticate with Google Cloud using service account key
                withCredentials([file(credentialsId: 'GKE_KEYS', variable: 'GKE_KEYS')]) {
                    sh 'gcloud auth activate-service-account --key-file=$GKE_KEYS'

                    // Setting the project
                    sh 'gcloud config set project $GKE_PROJECT_ID'

                    // Configure kubectl to use credentials for GKE cluster
                    sh 'gcloud container clusters get-credentials $GKE_CLUSTER_NAME --region $GKE_CLUSTER_REGION --project $GKE_PROJECT_ID'
                }

                script {
                    // Define the user and base URL from Jenkins parameters
                    def user = params.USER
                    def baseUrl = params.BASE_URL
                    def cpu = params.CPU
                    def ram = params.RAM
                    def password = params.PASSWORD
                    def storage = env.QUOTA_KB

                    // Set the deployment name and service name dynamically using the user parameter
                    // def deploymentName = "jupyter-notebooks-${user}"
                    // def serviceName = "jupyter-${user}"

                    // Dynamically replace the placeholders in the deployment YAML
                    sh """
                        sed -e 's|BASE_URL_PLACEHOLDER|${baseUrl}|g' \
                            -e 's|USER_PLACEHOLDER|${user}|g' \
                            -e 's|CPU|${cpu}|g' \
                            -e 's|RAM|${ram}|g' \
                            -e 's|QUOTA_KB|${storage}|g' \
                            -e 's|PASSWORD|${password}|g' \
                            deployment.yaml | kubectl apply -f -
                    """
                    sh """
                        sed -e 's|USER_PLACEHOLDER|${user}|g' \
                            service.yaml | kubectl apply -f -
                    """

                    // sh """
                    //     sed -e 's|BASE_URL_PLACEHOLDER|${baseUrl}|g' \
                    //         -e 's|USER_PLACEHOLDER|${user}|g' \
                    //         ingress.yaml | kubectl apply -f -
                    // """
                    
                }
            }
        }

    }
}
