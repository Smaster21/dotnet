pipeline {
    agent { label 's' }

    environment {
        NEXUS_URL = 'http://localhost:8081/repository/angular_vm/'
        NEXUS_CREDENTIALS_ID = 'nexus'
        GROUP_ID = 'angular'
        ARTIFACT_ID = 'angular-app' // Define ARTIFACT_ID
        VERSION = '1.0.0'
        FILE_EXTENSION = 'zip'
        SONARQUBE_PROJECT_KEY = 'Angular' 
        SONARQUBE_URL = 'http://localhost:9000'
        SONARQUBE_TOKEN = 'sqa_d43969d5576ce0ca814edd5404a33fc0c26223d5'
        
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/AYuS-V/Moosam_Ang.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Angular Project') {
            steps {
                sh 'ng build ' // Add --prod flag for production build
            }
        }
        
         stage('Test') {
            steps {
                    sh 'ng test'
                }
            }
        }

        
        stage('SonarQube Analysis') {
            steps {
                withEnv(["SONAR_HOST_URL=${env.SONARQUBE_URL}", "SONAR_LOGIN=${env.SONARQUBE_TOKEN}"]) {
                    sh """
                        npm install --save-dev sonar-scanner
                        npx sonar-scanner \
                            -Dsonar.projectKey=${env.SONARQUBE_PROJECT_KEY} \
                            -Dsonar.sources=src \
                            -Dsonar.host.url=${env.SONARQUBE_URL} \
                            -Dsonar.login=${env.SONARQUBE_TOKEN}
                    """
                }
            }
        }

        stage('Zip Dist Folder') {
            steps {
                script {
                    // Remove any existing zip files
                    sh "rm -rf /home/safumaster/Desktop/jenkins/workspace/angular/*.zip"
                    
                    // Generate the timestamp
                    def timestamp = sh(script: 'date +"%Y%m%d%H%M%S"', returnStdout: true).trim()
                    
                    // Create the zip file with the timestamped name
                    def distDir = '/home/safumaster/Desktop/jenkins/workspace/angular/dist'
                    def zipFile = "${env.ARTIFACT_ID}-${env.VERSION}-${timestamp}.${env.FILE_EXTENSION}"
                    def zipFilePath = "/home/safumaster/Desktop/jenkins/workspace/angular/${zipFile}"
                    
                    // Zip the dist folder
                    sh "cd ${distDir} && zip -r ${zipFilePath} ."
                    
                    // Set environment variables for the zip file path
                    env.ZIP_FILE_PATH = zipFilePath
                    env.ZIP_FILE = zipFile
                }
            }
        }
        
        stage('Upload to Nexus') {
            steps {
                script {
                    def nexusUrl = "${env.NEXUS_URL}${env.GROUP_ID.replace('.', '/')}/${env.VERSION}/${env.ZIP_FILE}"
                    
                    withCredentials([usernamePassword(credentialsId: env.NEXUS_CREDENTIALS_ID, passwordVariable: 'NEXUS_PASSWORD', usernameVariable: 'NEXUS_USERNAME')]) {
                        sh """
                            curl -u admin:admin --upload-file ${env.ZIP_FILE_PATH} ${nexusUrl}
                        """
                    }
                }
            }
        }

        stage('Download from Nexus') {
            steps {
                script {
                    // Remove any existing zip files
                    sh "rm -rf /home/safumaster/Desktop/jenkins/workspace/angular/*.zip"
                    
                    def nexusUrl = "${env.NEXUS_URL}${env.GROUP_ID.replace('.', '/')}/${env.VERSION}/${env.ZIP_FILE}"
                    
                    withCredentials([usernamePassword(credentialsId: env.NEXUS_CREDENTIALS_ID, passwordVariable: 'NEXUS_PASSWORD', usernameVariable: 'NEXUS_USERNAME')]) {
                        sh """
                            curl -u admin:admin -O ${nexusUrl}
                        """
                    }
                }
            }
        }
    }
}
