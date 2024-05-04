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
                sh "curl -L 'https://spectral-eu.checkpoint.com/latest/x/sh?dsn=$SPECTRAL_KEY' | sh"
            }
        }

        stage('Pre-Build Spectral Scan for Secrets,Misconfiguration and IaC') {
            steps {
                sh "$HOME/.spectral/spectral scan --engines secrets,oss --include-tags base,audit --ok"
            }
        }

        stage('Docker image Build and scan prep') {
            steps {
                sh 'docker build -t nilsujma/nginx .'
                sh 'docker save nilsujma/nginx -o nginx.tar'
            }
        }

        stage('ShiftLeft Container Image Scan Online') {
            steps {
                script {
                    try {
                        sh 'chmod +x shiftleft'
                        sh './shiftleft image-scan -t 180 -i nginx.tar -r 13172 -e 4b89d765-1dfd-4c19-bf26-a34374142d42'
                    } catch (Exception e) {
                        echo "ShiftLeft docker image scan failed."
                    }
                }
            }
        }

        stage('Shiftleft Container Image Scan Offline') {
            steps {
                script {
                    try {
                        sh 'chmod +x shiftleft'
                        sh './shiftleft image-scan -t 180 -i nginx.tar'
                    } catch (Exception e) {
                        echo "ShiftLeft docker Image scan failed."
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
