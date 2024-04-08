COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any
    tools {
        maven "maven3.9.6"
    }
    
    stages {
        stage ("git clone") {
            steps {
                git branch: 'main', url: 'https://github.com/JeanAmenyaglo/web-app.git'
            }
        }

    stage ("build with maven") {
        steps {
            sh "mvn clean"
        }
    }    
        
    stage ("testing with maven") {
        steps {
            sh "mvn test"
        }
    } 
    
    stage ("package with maven") {
        steps {
            sh "mvn package"
        }
    }

    stage('sonarQube analysis') {
            environment {
                ScannerHome = tool 'sonar5.0'
            }
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=jomacs"
                    }
                }
            }
        }

        stage ("upload to nexus") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'maven-web-application', classifier: '', file: '/var/lib/jenkins/workspace/first-pipeline-job/target/web-app.war', type: 'war']], credentialsId: 'nexus-id', groupId: 'com.mt', nexusUrl: '3.12.41.193:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'webapp-snapshot', version: '3.0.6-SNAPSHOT'
            }
        }

        stage ("deploy to UAT") {
            steps{
               deploy adapters: [tomcat9(credentialsId: 'tomcat-credential-2', path: '', url: 'http://3.134.80.210:8080')], contextPath: null, war: 'target/*.war' 
            }
        }
    
    } 

    post {
        always {
            slackSend channel: 'team-canada', 
                      color: COLOR_MAP[currentBuild.currentResult],
                      message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}"
        }
    }
}