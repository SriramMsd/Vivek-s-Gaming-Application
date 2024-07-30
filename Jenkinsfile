pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
   
    stages {
        stage('GIT CHECKOUT') {
            steps {
                git branch: 'main', url: 'https://github.com/VM2322/Vivek-s-Gaming-Application'
            }
        }
    
        stage('MAVEN COMPILE') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('MAVEN TEST') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('TRIVY FILESYSTEM SCAN') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
        stage('SONARQUBE ANALYSIS') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Gaming -Dsonar.projectKey=Gaming\
                            -Dsonar.java.binaries=. '''
                }
            }
        }
        
        stage('QUALITY GATE') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        
        stage('MAVEN BUILD') {
            steps {
                sh "mvn package"
            }
        }
        
        stage('BUILD & TAG DOCKER IMAGE') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t mygame12/gaming:latest ."
                    }
                }
            }
        }
    
        stage('IMAGE SCANNING') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html mygame12/gaming:latest"
            }
        }   
        
        stage('DOCKER DEPLOYMENT') {
            steps {
                sh "docker run -d -p 8001:8080 mygame12/gaming:latest"
            }
        }   
    }

    post {
        success {
            emailext (
                subject: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """\
Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' succeeded.

Build URL: ${env.BUILD_URL}
                """,
                to: 'srirammurugan98@gmail.com'
            )
        }
        failure {
            emailext (
                subject: "FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """\
Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' failed.

Build URL: ${env.BUILD_URL}

Please check the Jenkins build logs for more details.
                """,
                to: 'srirammurugan98@gmail.com'
            )
        }
    }
}
