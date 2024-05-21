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
                        sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=jomacs -Dsonar.host.url=http://54.210.195.228:9000/"
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
                    nexusUrl: 'http://54.144.135.136:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'webapp-snapshot', 
                    version: '3.0.13-SNAPSHOT'
            }
        }

        stage("Deploy to UAT") {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-id', path: '', url: 'http://54.198.165.151:8080/')], contextPath: null, war: 'target/*war' 
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

  
  stage("2. Build from Maven"){
    steps{
        sh "echo start building from Maven"
        sh "mvn clean package"
        sh "echo end of build"
    }  
  }
  
  stage("3. Code Scan with Sonarqube"){
    steps{
        sh "echo start of code scan using sonarqube"
        sh "mvn sonar:sonar"
        sh "echo end of code scan using sonarqube"
    }
  }
  
  stage("4. Deploy artifacts to Nexus"){
     steps{
         sh "echo Deploy to Nexus"
         sh "mvn deploy"
         sh "echo end Deploy to Nexus"
     }
  }
  
  stage("5. Deploy to Tomcat Server in UAT"){
    steps{
        sh "echo start deploying to tomcat server in UAT Environment"
        deploy adapters: [tomcat9(credentialsId: 'tomcard_credtial', path: '', url: 'http://18.118.1.16:9090/')], contextPath: null, war: 'target/*.war'
        sh "echo end deploying to tomcat server in UAT Environment"
    }
  }
  
  stage("6. Notification Alert"){
    steps{
        sh " echo start email notification to DevOps Team"
        emailext body: 'Declarative Project - Email Alert', subject: 'Declarative Project', to: 'info@jomacsit.com'
        sh " echo end email notification to DevOps Team"
    }
  }
  
 }
}
