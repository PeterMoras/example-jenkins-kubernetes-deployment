pipeline {
    agent none
    stages{
        stage('Setup'){
            agent {label 'built-in'}
            // environment {
            //     KUBECONFIG = credentials('local-kube-config')
            // }

            steps{
                withCredentials([usernamePassword(credentialsId: 'local-registry-creds', 
                                                            usernameVariable: 'DOCKER_USER', 
                                                            passwordVariable: 'DOCKER_PASS')]) {

                    
                    sh label: 'Setup registry secret in kubernetes', script:'''
                        kubectl create secret docker-registry local-registry-secret \
                            --docker-server=host.docker.internal:5000 \
                            --docker-username=${DOCKER_USER} \
                            --docker-password=${DOCKER_PASS} --dry-run=client -o yaml | kubectl apply -f -

                    '''
                }
                //ensureDockerSecretExists('docker-registry', 'local-registry-secret', 'host.docker.internal:5000')
            }
        }

        stage('Build & Push Image'){
            environment {
                LOCAL_REGISTRY = "host.docker.internal:5000"
                DOCKER_IMAGE = "${LOCAL_REGISTRY}/my-test-container"
                TAG = "${env.BUILD_NUMBER}"
                BUILD_PATH = "infrastructure/my-test-container"
                
            }

            agent {
                kubernetes {
                    //label 'kaniko-builder'
                    yamlFile 'jenkins/kaniko-builder.yml'
                }
            }

            steps {
                container('kaniko') {
                    sh """
                    /kaniko/executor \
                        --cache=true \
                        --context ${env.WORKSPACE}/${BUILD_PATH} \
                        --dockerfile ${env.WORKSPACE}/${BUILD_PATH}/Dockerfile \
                        --destination ${DOCKER_IMAGE}:${TAG} \
                        --destination ${DOCKER_IMAGE}:latest \
                        --insecure \
                        --skip-tls-verify \
                        --insecure-pull
                    
                    """
                }
            }
        }
    }

    



    
    post {
        success {
            echo "Successfully pushed ${DOCKER_IMAGE}:${TAG}"
        }
    }
}


// def ensureDockerSecretExists(String secretId, String k8sSecretName, String dockerServer = "https://index.docker.io/v1/") {
//     echo "Verifying Kubernetes secret: ${k8sSecretName}"
    
//     // Use the Jenkins agent's ability to run kubectl
//     withCredentials([usernamePassword(credentialsId: secretId, 
//                                      usernameVariable: 'U', 
//                                      passwordVariable: 'P')]) {
        
//         def checkCmd = "kubectl get secret ${k8sSecretName} --ignore-not-found"
//         def exists = sh(script: checkCmd, returnStdout: true).trim()

//         if (!exists) {
//             echo "Secret ${k8sSecretName} not found. Creating..."
//             sh """
//             kubectl create secret docker-registry ${k8sSecretName} \
//                 --docker-username='${U}' \
//                 --docker-password='${P}' \
//                 --docker-server='${dockerServer}'
//             """
//         } else {
//             echo "Secret ${k8sSecretName} already exists."
//         }
//     }
// }
