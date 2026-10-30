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
                // Alternative safe echo writing style
                sh '''
                    echo "FROM tomcat:9.0-jdk17-openjdk-slim" > Dockerfile
                    echo "RUN rm -rf /usr/local/tomcat/webapps/*" >> Dockerfile
                    echo "COPY target/*.war /usr/local/tomcat/webapps/ROOT.war" >> Dockerfile
                    echo "EXPOSE 8080" >> Dockerfile
                '''
                
                sh 'docker stop studentapp || true'
                sh 'docker rm studentapp || true'
                sh 'docker build -t student-data-image:latest .'
                sh 'docker run -d --name studentapp -p 8081:8080 student-data-image:latest'
            }
        }



    }
}
