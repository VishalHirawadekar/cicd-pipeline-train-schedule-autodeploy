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
                script{
                    def config = readYaml file: "train-schedule-kube-canary.yml"
                    config[1].spec.replicas = Integer.parseInt(env.CANARY_REPLICAS)
                    config[1].spec.template.spec.containers[0].image = env.DOCKER_IMAGE_NAME
                    writeYaml file: "train-schedule-kube-canary.yml", datas: config, overwrite: true
                }
                withKubeConfig([credentialsId: 'kubeconfig']) {
                   sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.22.15/bin/linux/amd64/kubectl"'
                   sh 'chmod u+x ./kubectl'
                   sh './kubectl apply -f train-schedule-kube-canary.yml'
               }
            }
        }
        stage('DeployToProduction') {
            environment { 
                CANARY_REPLICAS = 0
            }
            script{
                def config = readYaml file: "train-schedule-kube-canary.yml"
                config[1].spec.replicas = Integer.parseInt(env.CANARY_REPLICAS)
                config[1].spec.template.spec.containers[0].image = env.DOCKER_IMAGE_NAME
                writeYaml file: "train-schedule-kube-canary.yml", datas: config, overwrite: true
                def configProd = readYaml file: "train-schedule-kube.yml"
                configProd[1].spec.template.spec.containers[0].image = env.DOCKER_IMAGE_NAME
                writeYaml file: "train-schedule-kube.yml", datas: configProd, overwrite: true
                }
                withKubeConfig([credentialsId: 'kubeconfig']) {
                   sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.22.15/bin/linux/amd64/kubectl"'
                   sh 'chmod u+x ./kubectl'
                   sh './kubectl apply -f train-schedule-kube.yml'
               }
        }
    }
}
