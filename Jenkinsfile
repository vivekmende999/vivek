Project : devsecprjt
pipeline {
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME=tool 'sonar-vcube'
    }
    stages {
        
        stage ('clean Workspace'){
            steps{
                cleanWs()
            }
        }
        stage('SCM') {
            steps {
                git 'https://github.com/vivekmende999/jpetstore-6.git'
            }
        }
                stage('Compile'){
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test'){
            steps {
                sh 'mvn test'
            }
        }
        stage('package'){
            steps{
                sh 'mvn package'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petshop \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petshop '''
                }
            }
        }
        
        stage('Application Build'){
            steps{
                sh 'mvn package -X'
            }
        }
        
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'Dependency-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('nexus artfact uploader'){
            steps{
                nexusArtifactUploader artifacts: [[artifactId: 'mybatis-parent', classifier: '', file: ' /var/lib/jenkins/workspace/devsecprjt/target/jpetstore.war', type: 'war']], credentialsId: 'nexus', groupId: 'org.mybatis', nexusUrl: '13.50.226.92:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '6.1.1-SNAPSHOT'
            }
        }
        stage('tomcat'){
            steps{
                deploy adapters: [tomcat9(credentialsId: 'tomcat', path: '', url: 'http://13.53.184.27:8080/')], contextPath: null, war: '**/*.war'
            }
        }
    }
     post{
        always{
            emailext attachLog: true, body: '''BuildStatus:${currentBuild.result} 
            Build Number: ${currentBuild.number''', subject: 'Pipeline Status: $(currentBuild.result}', to: 'vivekmende99@gmail.com'
        }
    }
}
