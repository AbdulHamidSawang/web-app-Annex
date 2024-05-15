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

        stage('SonarQube Analysis') {
            environment {
                ScannerHome = tool name: 'sonar5.0', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
            }
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=jomacs -Dsonar.host.url=http://52.87.254.158:9000/"
                    }
                }
            }
        }

        stage("Upload to Nexus") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'maven-web-application', classifier: '', file: 'target/web-app.war', type: 'war']], 
                credentialsId: 'nexus-id', 
                groupId: 'com.mt', 
                nexusUrl: 'http://107.21.44.26:8081', 
                nexusVersion: 'nexus3', 
                protocol: 'http', 
                repository: 'webapp-snapshot', 
                version: '3.0.13-SNAPSHOT'
            }
        }

        stage("Deploy to UAT") {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-id', path: '', url: 'http://107.22.76.35:8080')], 
                contextPath: '', 
                war: 'target/web-app.war'
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
