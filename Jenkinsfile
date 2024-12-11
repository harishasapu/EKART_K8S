pipeline {
    agent any
    tools{
        maven 'maven3'
        jdk 'jdk17'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Gitcheckout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/harishasapu/EKART_K8S.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('SonarQube Analysis'){
            steps{
                withSonarQubeEnv('SonarQube'){
                    sh " $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=ekart -Dsonar.projectKey=ekart -Dsonar.java.binaries=. "
                }
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./app/backend --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs ."
            }
        }
        stage('Deploy To Nexus') {
            steps {
               withMaven(globalMavenSettingsConfig: 'mavenglobal', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true){                     
                sh " mvn deploy -DskipTests=true"
               }
            }
        }
    }
}
