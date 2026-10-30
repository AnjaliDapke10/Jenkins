pipeline {
    agent {
        label "jen-node1"
    }

    stages {
        stage('Pull') {
            steps {
                sh 'whoami'
                git branch: 'main', url: 'https://github.com/SurajBele/studentdata.git'
                echo "Git Pull Successful"
            }
        }

        stage('Build') {
            steps {
                sh '/opt/apache-maven-3.10.0/bin/mvn  package'
                echo "Build Successful"
            }
        }

        stage('Test') {
            steps {
                withSonarQubeEnv(installationName: 'Sonar-DB-Server', credentialsId: 'sonarqube-token') {
                sh '/opt/apache-maven-3.10.0/bin/mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar   -Dsonar.projectKey=Student-Data-Project'
                }
                echo "Test Successful"
            }
        }
        stage('quality gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
                echo "gate pass successful"
            }
        }

        stage('Deploy') {
            steps {
                // 1. Generate a clean Dockerfile using Tomcat's ROOT auto-deployment mechanism
                writeFile file: 'Dockerfile', text: '''
                    FROM tomcat:9.0-jdk17-openjdk-slim
                    
                    # Clear default webapps out
                    RUN rm -rf /usr/local/tomcat/webapps/*
                    
                    # Copy the war file as ROOT.war so it serves directly at the base URL (http://<IP>:8081/)
                    COPY target/*.war /usr/local/tomcat/webapps/ROOT.war
                    
                    EXPOSE 8080
                    CMD ["catalina.sh", "run"]
                '''
                
                // 2. Stop and clear out the old app container instances
                sh 'docker stop studentapp || true'
                sh 'docker rm studentapp || true'
                
                // 3. Rebuild image and launch container
                sh 'docker build -t student-data-image:latest .'
                sh 'docker run -d --name studentapp -p 8081:8080 student-data-image:latest'
                
                echo "Deploy Successful"
            }
        }

    }
}
