pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        SONAR_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git 'https://github.com/gashok13193/CICD-ekart.git'
            }
        }

        stage('Code Compile') {
            steps {
                sh "mvn clean compile"
            }
        }

        stage('Unit Testing') {
            steps {
                sh "mvn test -Dmaven.test.skip=true"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh ''' 
                    $SONAR_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=demo \
                    -Dsonar.projectName=demo \
                    -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage('OWASP Dependency') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dependency-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Build') {
            steps {
                sh "mvn package -Dmaven.test.skip=true"
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -Dmaven.test.skip=true"
                }
            }
        }

        stage('Docker Build and Tag') {
            steps {
                withDockerRegistry(credentialsId: 'fcaaed4a-5678-4547-b3dc-06eb5605ef83', url: 'https://index.docker.io/v1/') {
                    sh "docker build -t gashok13193/cicd-ekart:latest -f docker/Dockerfile ."
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh "trivy image gashok13193/cicd-ekart:latest > trivy-report.txt"
            }
        }

        stage('Docker Push') {
            steps {
                withDockerRegistry(credentialsId: 'fcaaed4a-5678-4547-b3dc-06eb5605ef83', url: 'https://index.docker.io/v1/') {
                    sh "docker push gashok13193/cicd-ekart:latest"
                }
            }
        }
    }
}
