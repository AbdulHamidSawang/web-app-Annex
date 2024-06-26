def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
    'UNSTABLE': 'warning',
    'ABORTED': 'warning'
]

pipeline {
    agent any
    tools {
        maven "maven3.9.6"
    }

    stages {
        stage("Git clone") {
            steps {
                git branch: 'main', url: 'https://github.com/AbdulHamidSawang/web-app-Annex.git'
            }
        }

        stage("Build with Maven") {
            steps {
                sh "mvn clean compile"
            }
        }

        stage("Testing with Maven") {
            steps {
                sh "mvn test"
            }
        }

        stage("Package with Maven") {
            steps {
                sh "mvn package"
            }
        }

        stage("SonarQube Analysis") {
            environment {
                ScannerHome = tool name: 'sonar5.0', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
            }
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=jomacs -Dsonar.host.url=http://54.146.65.66:9000/"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Upload to Nexus") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'maven-web-application', classifier: '', file: 'target/web-app.war', type: 'war']], 
                    credentialsId: 'nexus-id', 
                    groupId: 'com.mt', 
                    nexusUrl: 'http://54.87.216.82:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'webapp-snapshot', 
                    version: '3.0.13-SNAPSHOT'
            }
        }

        stage("Deploy to UAT") {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-id', path: '', url: 'http://54.82.207.82:8080')], 
                    contextPath: '', 
                    war: 'target/web-app.war'
            }
        }
    }

    post {
        always {
            script {
                def color = COLOR_MAP.get(currentBuild.currentResult, 'warning')
                slackSend channel: '@Abdul-Hamid Sawang', 
                          color: color,
                          message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\nMore info at ${env.BUILD_URL}"
            }
        }
    }
}
