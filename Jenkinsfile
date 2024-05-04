pipeline {
    agent any

    environment {
        // Use Jenkins credentials binding
        CHKP_CLOUDGUARD_ID = credentials('CHKP_CLOUDGUARD_ID')
        CHKP_CLOUDGUARD_SECRET = credentials('CHKP_CLOUDGUARD_SECRET')
        SHIFTLEFT_REGION = "us1"
        SPECTRAL_DSN = credentials('SPECTRAL_DSN')
        SPECTRAL_KEY = credentials('SPECTRAL_KEY')
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
                sh "$HOME/.spectral/spectral scan --engines secrets,oss --include-tags base,audit --ok"
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

        stage('Post-Build Spectral Scan for Secrets,Misconfiguration and IaC') {
            steps {
                sh "$HOME/.spectral/spectral scan --engines secrets,oss --include-tags base,audit --ok"
            }
        }
    }
}
