pipeline {
    agent none

    environment{
        // REGISTRY_URL = 'nexus.local'
        IMAGE_NAME = 'my-test-container'
    }

    stages{


        stage('Build & Push Image'){
            environment {
                LOCAL_REGISTRY = "nexus-nexus3.nexus:8082"
                DOCKER_IMAGE = "${LOCAL_REGISTRY}/${IMAGE_NAME}"
                TAG = "${env.BUILD_NUMBER}"
                BUILD_PATH = "infrastructure/${IMAGE_NAME}"
                
            }

            agent {
                kubernetes {
                    //label 'kaniko-builder'
                    yamlFile 'kubernetes/kaniko-builder.yml'
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
