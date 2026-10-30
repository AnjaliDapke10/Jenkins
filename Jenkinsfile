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
                // 1. Dynamically generate the Dockerfile in the Jenkins workspace directory
                writeFile file: 'Dockerfile', text: '''
                    FROM tomcat:9.0-jdk21-openjdk-slim
                    RUN rm -rf /usr/local/tomcat/webapps/*
                    COPY target/*.war /usr/local/tomcat/webapps/studentdata.war
                    RUN sed -i "/<\/web-app>/i <welcome-file-list><welcome-file>index.jsp<\/welcome-file><\/welcome-file-list>" /usr/local/tomcat/conf/web.xml
                    EXPOSE 8080
                    CMD ["catalina.sh", "run"]
                '''
                
                // 2. Clear old running instances to avoid port conflicts
                sh 'docker stop studentapp || true'
                sh 'docker rm studentapp || true'

                
                // 3. Build and launch the container
                sh 'docker build -t student-data-image:latest .'
                sh 'docker run -d --name studentapp -p 8081:8080 student-data-image:latest'
                
                echo "Deploy Successful"
            }
        }
    }
}
