pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Fetch Code') {
            steps {
               git branch: 'main', credentialsId: 'git-login', url: 'https://github.com/iamsaurav-karki/FullStack-Blogging-App.git'
            }
        }
        
         stage('Complilation of Code') {
            steps {
                sh "mvn compile"
            }
        }
        
         stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        
         stage('Trivy scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }
        
        
          stage('SonarQube analysis') {
            steps {
                 withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                     -Dsonar.projectKey=Blogwebsite \
                     -Dsonar.projectName=Blogwebsite \
                     -Dsonar.java.binaries=target  \
                     -Dsonar.sources=.
                    '''
            }
          }
        }
        
        
         stage('Build') {
            steps {
                sh "mvn package"
            }
        }
        
         stage('Publish Artifacts') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                sh "mvn deploy"
            }
            }
        }
        
         stage('Docker Build and Tag') {
            steps {
                script {
                         withDockerRegistry(credentialsId: 'dockerlogin', toolName: 'docker') {
                
                    sh "docker build -t sauravkarki/bloggingapp:latest ."
                  }
                }
            
            }
        }
        
          stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image.html sauravkarki/bloggingapp:latest"
            }
        }
        
        
         stage('Docker Push Image') {
            steps {
                script {
                         withDockerRegistry(credentialsId: 'dockerlogin', toolName: 'docker') {
                
                    sh "docker push sauravkarki/bloggingapp:latest "
                  }
                }
            
            }
        }
        
         stage('Docker Deploy To Dev') {
            steps {
                script{
                     withDockerRegistry(credentialsId: 'dockerlogin', toolName: 'docker') {
                    sh "docker run -d -p 8060:8080 sauravkarki/bloggingapp:latest"
                }
            }
            }
        }

        
    }
    
}
