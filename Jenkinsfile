pipeline {
    agent {
        label "jen-node1"
    }

    stages {
        stage('Pull') {
            steps {
                git branch: 'main', url: 'https://github.com'
                echo "Git Pull Successful"
            }
        }

        stage('Build') {
            steps {
                sh '/opt/apache-maven-3.10.0/bin/mvn clean package'
                echo "Build Successful"
            }
        }

        stage('Test') {
            steps {
                withSonarQubeEnv(installationName: 'Sonar-DB-Server', credentialsId: 'sonarqube-token') {
                    sh '/opt/apache-maven-3.10.0/bin/mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=Student-Data-Project'
                }
                echo "Test Successful"
            }
        }

        stage('quality gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
                echo "gate pass successful"
            }
        }

        stage('Deploy') {
            steps {
                // 1. Generate the Dockerfile without complex escape slashes
                writeFile file: 'Dockerfile', text: '''
                    FROM tomcat:9.0-jdk17-openjdk-slim
                    RUN rm -rf /usr/local/tomcat/webapps/*
                    COPY target/*.war /usr/local/tomcat/webapps/studentdata.war
                    
                    # Create a default ROOT application to automatically redirect traffic
                    RUN mkdir /usr/local/tomcat/webapps/ROOT
                    RUN echo '<% response.sendRedirect(request.getContextPath() + "/studentdata/index.jsp"); %>' > /usr/local/tomcat/webapps/ROOT/index.jsp
                    
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
