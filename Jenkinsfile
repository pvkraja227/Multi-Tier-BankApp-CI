pipeline { 
    agent any

    parameters {
        string(name: 'DOCKER_TAG', defaultValue: 'latest', description: 'Docker tag')
    }
    
    tools {
        maven 'maven3'
    }
    environment {
        SONAR_HOME= tool 'sonar-scanner'
    }
    
    stages {

        stage('clean workspace') {
            steps {
                cleanWs() // clean the workspace
            }
        }

        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/pvkraja227/Multi-Tier-BankApp-CI.git'        
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=Bank-App -Dsonar.projectKey=Bank-App -Dsonar.java.binaries=target"
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
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                sh "mvn deploy -DskipTests=true"
                }
                
            }
        }
        
        stage('Docker Build & Tag') {
            steps {
                script {
                withDockerRegistry(credentialsId: 'docker-cred') {
                
                sh "docker build -t rajapvk23/Banking-App:${params.DOCKER_TAG} ."    
                }
                }
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image.html rajapvk23/Banking-App:${params.DOCKER_TAG}"
            }
        }
        
        stage('Docker Push') {
            steps {
                script {
                withDockerRegistry(credentialsId: 'docker-cred') {
                
                sh "docker push rajapvk23/Banking-App:${params.DOCKER_TAG}"    
                }
                }
            }
        }

        stage ('Update YAML Manifest in Other Repo') {
            steps {
                script {
                withCredentials([gitUsernamePassword(credentialsId: 'git-cred', gitToolName: 'Default')]) {
                    sh '''
                        # clone the repo
                        git clone https://github.com/pvkraja227/Multi-Tier-BankApp-CD.git
                        cd Multi-Tier-BankApp-CD

                        # List files to confirm the presence of bankapp-ds.yml
                        ls -l bankapp

                        # get the absolute path for the current directory
                        repo_dir=$(pwd)

                        # use the absolute path for sed
                        sed -i 's|image: rajapvk23/Banking-App:.*|image: rajapvk23/Banking-App:'${DOCKER_TAG}'|' ${repo_dir}/bankapp/bankapp-ds.yml
                    '''

                    // confirm the change
                    sh '''
                        echo "updated YAML file contents:"
                        cat Multi-Tier-BankApp-CD/bankapp/bankapp-ds.yml
                    '''

                    // configure git for committing changes and pushing
                    sh '''
                        cd Multi-Tier-BankApp-CD # ensure you are inside the cloned repo
                        git config user.email "venkat.klce227@gmail.com"
                        git config user.name "pvkraja227"
                    '''

                    // commit and push the updated yaml file back to the other repository
                    sh '''
                        cd Multi-Tier-BankApp-CD
                        ls
                        git add bankapp/bankapp-ds.yml
                        git commit -m "Update image tag to ${DOCKER_TAG}"
                        git push origin main
                    '''
                }
                }
            }
        }
    }
}

