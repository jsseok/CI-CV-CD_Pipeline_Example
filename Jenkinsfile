pipeline {
    agent any
    
    environment {
        DOCKER_HUB_USER = 'jsseok'
        DOCKER_HUB_PASS = 'dckr_pat_RqZTw6eodJwkHXiMnd3Cn9DnD0A'
        KUBECONFIG = credentials('k8s-config')
    }

    stages {
        stage('Pipeline Artifact Directory Setting') {
            steps {
                echo 'directory setting'
                sh 'mkdir -p jenkins/artifact'
            }
        }
        stage('Requirement Analysis') {
            steps {
                echo 'requirement analysis'
                script {
                    if (fileExists('jenkins/user_requirement.txt')) {
                        sh 'cat jenkins/user_requirement.txt'
                        sh 'python3 jenkins/analyze_req.py'
                        sh 'cat jenkins/artifact/list_of_features.txt'
                    } else {
                        echo 'user_requirement.txt not found'
                    }
                }
            }
        }
        stage('ROS Node Integration') {
            steps {
                echo 'ros node inetgration'
                script {
                    if (fileExists('jenkins/artifact/list_of_features.txt')) {
                        sh 'python3 jenkins/integrate_node.py'
                        sh 'cat list_of_nodes.yaml'
                    } else {
                        echo 'list_of_features.txt not found'
                    }
                }
            }
        }
        stage('Container Packaging') {
            steps {
                echo 'container packaging'
                script {
                    // if (fileExists('jenkins/artifact/list_of_nodes.yaml')) {
                    //     echo 'execute some commands'
                    // } else {
                    //     echo 'list_of_nodes.yaml not found'
                    // }
                    sh '''
                    echo $DOCKER_HUB_PASS | docker login -u $DOCKER_HUB_USER --password-stdin

                    if docker context inspect buildctx > /dev/null 2>&1; then
                        docker context rm buildctx
                    fi
                    docker context create buildctx --docker "host=tcp://129.254.174.129:32375"

                    if docker buildx inspect builderx > /dev/null 2>&1; then
                        docker buildx rm builderx
                    fi
                    docker buildx create --name builderx --use buildctx
                    '''
                    
                    if (fileExists('container/container_build.sh')) {
                        sh 'sh ./container/container_build.sh'
                    } else {
                        echo 'container_build.sh not found'
                    }
                }
            }
        }
        // stage('Container Push') {
        //     steps {
        //         echo 'container push'
        //     }
        // }
        stage('Deploy apps to k8s cluster') {
            steps {
                echo 'Deploy to k8s cluster'
                sh 'kubectl apply -f ./container/k8s_deployment.yaml'
            }
        }
    }
    post {
        always {
            echo "Pipeline completed."
        }
    }
}
