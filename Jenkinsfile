pipeline {
    agent {
      label 'Slave'
    }
    
    environment {
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
    }
    
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "M3"
    }
    
    stages {
        stage('Checkout GIT') {
            steps {
                git branch: 'panda_application', url:'https://github.com/mareckii83/panda_application.git'
            }
        }

        stage('Clear running apps') {
            steps {
                // Run Maven on a Unix agent.
                sh 'docker rm -f pandaapp || true'
                sh "echo cleaning"
            }
        }
        stage('Build and Junit') {
            steps {
                // Run Maven on a Unix agent.
                sh "mvn clean install"
            }
        }
        stage('Build Docker image'){
            steps {
                sh "echo build docker"
                sh "mvn package -Pdocker"
            }
        }
        stage('Run Docker app') {
            steps {
                sh "echo run docker"
    			sh "docker run -d -p 0.0.0.0:8080:8080 --name pandaapp -t ${IMAGE}:${VERSION}"
            }
        }
        stage('Test Selenium') {
            steps {
                sh "echo run selenium"
				sh "mvn test -Pselenium"
            }
        }
        stage('Deploy jar to artifactory') {
            steps {
                configFileProvider([configFile(fileId: '9d1ed313-ea70-4fa9-9934-7108c53eca75', variable: 'MAVEN_GLOBAL_SETTINGS')]) {
                    sh "mvn -gs $MAVEN_GLOBAL_SETTINGS deploy -Dmaven.test.skip=true -e"
                }
            } 
            post {
                always { 
                    sh 'docker stop pandaapp'
                    deleteDir()
                }
            }
        }
        
    }
}
