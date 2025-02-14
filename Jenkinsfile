pipeline {
    agent any
    
    tools{
        jdk "jdk17"
        maven "maven3"
    }
    environment {
        SCANNER_HOME= tool "sonar-scanner"
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/akshayarsude1997/Ekart-java-project.git'
            }
        }
        stage('compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('unit test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        stage('sonarqube analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART \
                   -Dsonar.java.binaries=. '''
                }
            }
        }
        // stage('owasp dp check') {
        //     steps {
        //         dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DP'
        //         dependencyCheckPublisher pattern: '\'**/dependency-check-report.xml\''
        //     }
        // }
        stage('build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        stage('Deploy to nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-one', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        stage('docker image build') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                    sh "docker build -t akshayarsude1997/ekart -f docker/Dockerfile ."
                }
                }
            }
        }
        stage('trivy image scane') {
            steps {
                sh "trivy image akshayarsude1997/ekart > trivy-report.txt"
            }
        }
        
        stage('docker image push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                    sh "docker push akshayarsude1997/ekart"
                }
                }
            }
        }
        
        
    }
}
