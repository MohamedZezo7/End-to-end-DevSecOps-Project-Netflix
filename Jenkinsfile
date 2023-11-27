pipeline{
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        REGISTEY_NAME = 'mohamedabdelaziz10'
        REPO_NAME = 'netflix'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages{
        stage('Clean Workspace'){
            steps{
                CleanWs()
            }

        }
        stage('checkout from Git'){
            steps{

                git branch: 'main', url: 'https://github.com/MohamedZezo7/End-to-end-DevSecOps-Project-Netflix'

            }

        }
        stage('Static Code Analysis'){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                  }
               }
        }


        stage('Quality Gate'){
            steps{
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-id' 
            }
        }
        stage('Install Dependencies'){
            steps{
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN'){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
            
        }
        stage('Trivy fs Scan'){
            steps{
                sh "trivy fs . > trivyfs.txt"
            }
            
        }

        stage('Docker Build & Push '){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub-id' , toolName: 'docker') {
                        sh "docker build --build-arg TMDB_V3_API_KEY=8761d86e8e6b4b37ef0085e2db8887ae -t netflix . "
                        sh "docker tag netflix ${REGISTRY_NAME}/${REPO_NAME}:${IMAGE_TAG}"
                        sh "docker push ${REGISTRY_NAME}/${REPO_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Trivy'){
            steps{
                sh "trivy image ${REGISTRY_NAME}/${REPO_NAME}:${IMAGE_TAG} > trivyimage.txt"
            }
        }
        stage('Deploy to Container'){
            steps{

                sh "docker run -d --name netflix -p 80:80 ${REGISTRY_NAME}/${REPO_NAME}:${IMAGE_TAG}"

            }
            
        }



    }
}
}