//Declarative Pipeline
def VERSION='1.0.0'
pipeline {
    agent none
    // tools {
	//  maven 'apache-maven-3.6.3'
    // }
    environment {
        PROJECT = "WELCOME TO DEVOPS B32 BATCH - Jenkins Class"
        AZ_SUB_ID = "9ce91e05-4b9e-4a42-95c1-4385c54920c6"
        AZ_TEN_ID = "2b387c91-acd6-4c88-a6aa-c92a96cab8b1"
    }
    stages {
        stage("Dev Tools Verification") {
            when {
                branch 'development'
            }
            agent { label 'DEV' }
            steps {
                sh "mvn --version"
                sh "java -version"
                sh "terraform version"
                sh "packer version"
                sh "trivy --version"

            }
        }
        stage('Dev Sonarqube SAST') {
            when {
                branch 'development'
            }
            agent { label 'DEV' }
            steps { 
                withSonarQubeEnv('DevOpsB32-SonarQube-DEV'){
                     sh "mvn clean verify sonar:sonar \
                     -Dsonar.projectKey=spring-boot-app-dev \
                     -Dsonar.projectName=spring-boot-app-dev \
                     -Dsonar.host.url=http://sonarqube.awscloudclass.xyz:9000"
                }

            }
        }
        stage("Dev Quality gate") {
            when {
                branch 'development'
            }
            steps {
                waitForQualityGate abortPipeline: true
            }
        }
        stage('Dev mvn clean') {
            when {
                branch 'development'
            }
            agent { label 'DEV' }
            steps { 
                sh "mvn clean"
                // exit 1
            }
        }
        stage('Dev mvn test') {
            when {
                branch 'development'
            }
            agent { label 'DEV' }
            steps { 
                sh "mvn test"
            }
        }
        stage('Dev mvn package & install') {
            when {
                branch 'development'
            }
            agent { label 'DEV' }
            steps { 
                sh "mvn versions:set -DnewVersion=Dev-1.0.${BUILD_NUMBER}"
                sh "mvn package install"
                sh "rm -rf /home/ubuntu/.m2/settings.xml"
                sh "cp dev-settings.xml /home/ubuntu/.m2/settings.xml"
            }
        }
        stage('Dev mvn package & deploy') {
            when {
                branch 'development'
            }
            agent { label 'DEV' }
            steps {
                sh "mvn versions:set -DnewVersion=Dev-1.0.${BUILD_NUMBER}" 
                sh "mvn package deploy"
            }
        }
        stage('Dev docker build') {
            when {
                branch 'development'
            }
            agent { label 'DEV' }
            steps { 
                sh "sudo docker build -t sreeharshav/devopsb32spring:$BUILD_NUMBER ."
            }
        }
        stage('Dev Trivy Scan') {
            when {
                branch 'development'
            }
            agent { label 'DEV' }
            steps {
                        sh 'curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl > html.tpl'
                        sh 'mkdir -p reports && rm -rf reports/dev_trivy_report.html' 
                        sh """trivy image sreeharshav/devopsb32spring:$BUILD_NUMBER --security-checks vuln --exit-code 1 --severity CRITICAL --timeout 15m --format template --template \"@html.tpl\" --output reports/dev_trivy_report.html  """
                        
                        // Publish Dev trivy HTML findings Report
                        publishHTML (target : [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'reports',
                        reportFiles: 'dev_trivy_report.html',
                        reportName: 'Trivy Scan',
                        reportTitles: 'Trivy Scan'
                        ])
                    }

            }
        stage('Dev Deploy Docker Image') {
            when {
                branch 'development'
            }
            agent { label 'DEV' }
            steps { 
                sh "sudo docker stop springbootapp || sudo docker ps"
                sh "sudo docker run --rm -dit --name springbootapp -p 8080:8080 sreeharshav/devopsb32spring:$BUILD_NUMBER"
            }
        }
        stage('Dev Validate Deployment') {
            when {
                branch 'development'
            }
            options {
               timeout(time: 3, unit: 'MINUTES') 
            }
            agent { label 'DEV' }
            steps { 
                sh "sleep 30 && curl http://ec2-44-198-61-11.compute-1.amazonaws.com:8080 || exit 1"
            }
        }
        stage ('Dev DAST') {
            when {
                branch 'development'
            }
          options {
            timeout(time: 5, unit: 'MINUTES') 
          }
          agent { label 'DEV' }  
          steps {
             sh 'sudo docker run -t owasp/zap2docker-stable zap-baseline.py -t http://ec2-44-198-61-11.compute-1.amazonaws.com:8080 || true'
            }
        }
//-----------------------------PRODUCTION---------------
        stage("PROD Tools Verification") {
            when {
                branch 'production'
            }
            agent { label 'PROD' }
            steps {
                sh "mvn --version"
                sh "java -version"
                sh "terraform version"
                sh "packer version"
            }
        }
        stage('PROD Sonarqube SAST') {
            when {
                branch 'production'
            }
            agent { label 'PROD' }
            steps { 
                withSonarQubeEnv('DevOpsB32-SonarQube-PROD'){
                     sh "mvn clean verify sonar:sonar \
                     -Dsonar.projectKey=spring-boot-app-prod \
                     -Dsonar.projectName=spring-boot-app-prod \
                     -Dsonar.host.url=http://sonarqube.awscloudclass.xyz:9000"
                }

            }
        }
        stage("PROD Quality gate") {
            when {
                branch 'production'
            }
            steps {
                waitForQualityGate abortPipeline: true
            }
        }
        stage('PROD mvn clean') {
            when {
                branch 'production'
            }
            agent { label 'PROD' }
            steps { 
                sh "mvn clean"
                // exit 1
            }
        }
        stage('PROD mvn test') {
            when {
                branch 'production'
            }
            agent { label 'PROD' }
            steps { 
                sh "mvn test"
            }
        }
        stage('PROD mvn package & install') {
            when {
                branch 'production'
            }
            agent { label 'PROD' }
            steps {
                sh "mvn versions:set -DnewVersion=Prod-${BUILD_NUMBER}" 
                sh "mvn package install"
                sh "rm -rf /home/ubuntu/.m2/settings.xml"
                sh "cp dev-settings.xml /home/ubuntu/.m2/settings.xml"
            }
        }
        stage('PROD mvn package & deploy') {
            when {
                branch 'production'
            }
            agent { label 'PROD' }
            steps { 
                sh "mvn versions:set -DnewVersion=Prod-${BUILD_NUMBER}"
                sh "mvn package deploy"
            }
        }
        stage('PROD docker build') {
            when {
                branch 'production'
            }
            agent { label 'PROD' }
            steps { 
                sh "sudo docker build -t sreeharshav/devopsopsb32springprod:$BUILD_NUMBER ."
            }
        }
        stage('Prod Trivy Scan') {
            when {
                branch 'production'
            }
            agent { label 'PROD' }
            steps {
                        sh 'curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl > html.tpl'
                        sh """sudo trivy image sreeharshav/devopsopsb32springprod:$BUILD_NUMBER --security-checks vuln --exit-code 0 --severity CRITICAL --timeout 15m --format template --template \"@html.tpl\" --output prod_trivy_report.html  """

                        // Publish Prod trivy HTML findings Report
                        publishHTML target : [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportFiles: 'prod_trivy_report.html',
                        reportName: 'Trivy Scan',
                        reportTitles: 'Trivy Scan'
                        ]
                    }
                }
        stage('PROD Deploy Docker Image') {
            when {
                branch 'production'
            }
            agent { label 'PROD' }
            steps { 
                sh "sudo docker stop springbootapp || sudo docker ps"
                sh "sudo docker run --rm -dit --name springbootapp -p 8080:8080 sreeharshav/devopsopsb30springprod:$BUILD_NUMBER"
            }
        }
        stage('PROD Validate Deployment') {
            when {
                branch 'production'
            }
            options {
               timeout(time: 3, unit: 'MINUTES') 
            }
            agent { label 'PROD' }
            steps { 
                sh "sleep 30 && curl http://ec2-3-231-59-39.compute-1.amazonaws.com:8080 || exit 1"
            }
        }
        stage ('PROD DAST') {
            when {
                branch 'production'
            }
          options {
            timeout(time: 5, unit: 'MINUTES') 
          }
          agent { label 'PROD' }  
          steps {
             sh 'sudo docker run -t owasp/zap2docker-stable zap-baseline.py -t http://ec2-3-231-59-39.compute-1.amazonaws.com:8080 || true'
            }
        }
    }
    post {
    success {
        slackSend(color: 'good', message: "Pipeline Successfull: ${env.JOB_NAME} ${env.BUILD_NUMBER} ${env.BUILD_URL}") 
    }
    failure {
        slackSend(color: 'danger', message: "Pipeline Failed: ${env.JOB_NAME} ${env.BUILD_NUMBER} ${env.BUILD_URL}") 
    }
    aborted {
        slackSend(color: 'warning', message: "Pipeline Aborted: ${env.JOB_NAME} ${env.BUILD_NUMBER} ${env.BUILD_URL}")
    }
    always {
        echo "I always run."
    }
    }

}


