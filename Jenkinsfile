pipeline {
    agent {
        node {
            label 'Slave-node1'
        }
    }
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "vhirawadekar/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh 'sudo ./gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
//         stage('Build Docker Image') {
//             steps {
//                 script {
//                     app = docker.build(DOCKER_IMAGE_NAME)
//                     app.inside {
//                         sh 'echo Hello, World!'
//                     }
//                 }
//             }
//         }
//         stage('Push Docker Image') {
//             steps {
//                 script {
//                     docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
//                         app.push("${env.BUILD_NUMBER}")
//                         app.push("latest")
//                     }
//                 }
//             }
//         }
        stage('CanaryDeploy') {
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                   sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.22.15/bin/linux/amd64/kubectl"'
                   sh 'chmod u+x ./kubectl'
                   sh './kubectl get pods'
               }
            }
        }
        stage('DeployToProduction') {
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
