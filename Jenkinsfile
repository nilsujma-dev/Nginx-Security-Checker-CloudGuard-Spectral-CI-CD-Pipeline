pipeline {
    agent any

    environment {
        // Use Jenkins credentials binding
        CHKP_CLOUDGUARD_ID = credentials('CHKP_CLOUDGUARD_ID')
        CHKP_CLOUDGUARD_SECRET = credentials('CHKP_CLOUDGUARD_SECRET')
        SHIFTLEFT_REGION = "us1"
        SPECTRAL_DSN = credentials('SPECTRAL_DSN')
        SPECTRAL_KEY = credentials('SPECTRAL_KEY')
        DOCKER_USERNAME = credentials('DOCKER_USERNAME')
        DOCKER_PASSWORD = credentials('DOCKER_PASSWORD')
        GCR_JSON_KEY = credentials('GCR_JSON_KEY')
        KUBECONFIG = credentials('jenkins-gke-kubeconfig')


         }

    stages {
        stage('Clone Github repository') {
            steps {
                checkout scm
            }
        }

        stage('install Spectral') {
            steps {
                sh "curl -L 'https://spectral-us.checkpoint.com/latest/x/sh?key=$SPECTRAL_KEY' | sh"
            }
        }

        stage('Pre-Build Spectral Scan for Secrets,Misconfiguration and IaC') {
            steps {
                sh "$HOME/.spectral/spectral scan --engines secrets,iac --include-tags base,audit,iac --ok"
            }
        }

        stage('Pre-Build Spectral Scan for OSS') {
            steps {
                sh "$HOME/.spectral/spectral scan --engines oss --ok"
            }
        }

        stage('Docker image Build and scan prep') {
            steps {
                sh 'docker build -t scb/nginx .'
                sh 'docker save scb/nginx -o nginx.tar'
            }
        }

        stage('ShiftLeft Container Image Scan Online') {
            steps {
                script {
                    try {
                        sh 'chmod +xxx shiftleft'
                        sh 'env'
                        sh 'pwd'
                        sh 'ls -la'
                        sh './shiftleft image-scan -e 5c43dd24-8b60-4a44-b012-d1d3bc4b36cb -i nginx.tar'
                    } catch (Exception e) {
                        echo "ShiftLeft docker image scan failed."
                    }
                }
            }
        }

        stage('Docker Login and Push Image') {
            steps {
                script {
                    sh 'echo $DOCKER_PASSWORD | docker login scb-harbor.cpdevsecops.net --username $DOCKER_USERNAME --password-stdin'
                    sh 'docker tag scb/nginx scb-harbor.cpdevsecops.net/scb-playground/nginx-web:latest'
                    sh 'docker push scb-harbor.cpdevsecops.net/scb-playground/nginx-web:latest'

                        }
                    }
        }

        stage('Docker Login and Push Image to GCR') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'GCR_JSON_KEY', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh 'cat \$GOOGLE_APPLICATION_CREDENTIALS | docker login -u _json_key --password-stdin https://asia.gcr.io'
                        sh 'docker tag scb/nginx asia.gcr.io/chkp-gcp-sales-nilsu-box/scb-playground/nginx-web:latest'
                        sh 'docker push asia.gcr.io/chkp-gcp-sales-nilsu-box/scb-playground/nginx-web:latest'
                    }
                }
            }
        }


        stage('Deploy to GKE') {
            steps {
                script {
                    // Use the kubeconfig file uploaded to Jenkins
                    withCredentials([file(credentialsId: 'jenkins-gke-kubeconfig', variable: 'KUBECONFIG')]) {
                        // Commands to apply Kubernetes configurations, assuming you have deployment.yaml in your repository
                        sh "kubectl apply -f deployment.yaml --kubeconfig=\$KUBECONFIG"
                        // Add other kubectl commands as needed
                    }
                }
            }
        }

    }
}
