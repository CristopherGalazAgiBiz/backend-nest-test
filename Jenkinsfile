pipeline {
    agent any
    environment{
        NPM_CONFIG_CACHE= "${WORKSPACE}/.npm"
        dockerImagePrefix = "us-west1-docker.pkg.dev/lab-agibiz/docker-repository"
        registry = "https://us-west1-docker.pkg.dev"
        registryCredentials = "gcp-registry"
    }
    stages{
        stage ("proceso de builds y test") {
            agent {
                docker{
                    image 'node:22'
                    reuseNode true
                }
            }
            stages{
                stage ('Instalacion de dependencias'){
                    steps {
                        sh 'npm ci'
                    }
                }
                stage ('ejecucion de pruebas'){
                    steps {
                        sh 'npm run test:cov'
                    }
                }
                stage ('construccion de la aplicacion'){
                    steps {
                        sh 'npm run build'
                    }
                }
            }
        }
        stage ('build y push de imagen docker'){
            steps {
                script{
                    docker.withRegistry("${registry}", registryCredentials){
                        sh "docker build -t backend-nest-test-cgc ."
                        sh "docker tag backend-nest-test-cgc ${dockerImagePrefix}/backend-nest-test-cgc"
                        sh "docker tag backend-nest-test-cgc ${dockerImagePrefix}/backend-nest-test-cgc:${BUILD_NUMBER}"
                        sh "docker push ${dockerImagePrefix}/backend-nest-test-cgc"
                        sh "docker push ${dockerImagePrefix}/backend-nest-test-cgc:${BUILD_NUMBER}"
                    }
                }
            }
        }
        stage ("Actualización de kubernetes"){
            agent {
                docker {
                    image 'alpine/k8s:1.30.2'
                    reuseNode true
                }
            }
            steps {
                withKubeConfig([credentialsId: 'gcp-kubeconfig']){
                    sh "kubectl -n test-lab-cgc set image deployments/backend-nest-test-cgc backend-nest-test-cgc=${dockerImagePrefix}/backend-nest-test-cgc:${BUILD_NUMBER}"
                }
            }
        }
    }
}