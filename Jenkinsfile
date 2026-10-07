// pipeline written by vamsi krishna kakani on oct:06/2026

pipeline {
    agent {
        node {
            label "prod"
        }
    }
    tools {
        maven "mymaven"
    }
    stages {
        stage ("Code") {
            steps {
                git 'https://github.com/kakanivamsikrishna/docker-e2e-app.git'
            }
        }
        stage ("Build") {
            steps {
                sh "mvn clean install"
            }
        }
        stage ("CQA") {
            steps {
                withSonarQubeEnv("mysonar") {
                    sh " mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=myproject"
                }
            }
        }
        stage ("Quality check") {
            steps {
                waitForQualityGate abortPipeline: true, credentialsId: 'sonar'
            }
        }
        stage ("Artifacts") {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    nexusArtifactUploader artifacts: [[artifactId: pom.artifactId, classifier: '', file: 'target/vprofile-v2.war', type: 'war']], credentialsId: 'nexus', groupId: pom.groupId, nexusUrl: '77.113.39.139:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'myrepo', version: pom.version
                }
            }
        }
        stage ("Image creation") {
            steps {
                sh "cp -r target Docker-app"
                sh "docker build -t vamsikrishnakakani/test-repo:javaimage Docker-app"
                sh "docker build -t vamsikrishnakakani/test-repo:dbimage Docker-db"
            }
        }
        stage ("Trivy scan") {
            steps {
                sh "trivy image vamsikrishnakakani/test-repo:javaimage >> javareport.txt"
                sh "trivy image vamsikrishnakakani/test-repo:dbimage >> dbreport.txt"
            }
        }
        stage ("Dockerhub Registry") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        sh "docker push vamsikrishnakakani/test-repo:javaimage"
                        sh "docker push vamsikrishnakakani/test-repo:dbimage"
                    }
                }
            }
        }
        stage ("Deploy") {
            steps {
                sh "docker stack deploy myapp --compose-file=compose.yml"
            }
        }
    }
}
