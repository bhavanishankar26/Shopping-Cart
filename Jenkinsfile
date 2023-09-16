pipeline {
    agent any
    
    tools{
        jdk 'jdk11'           // Define these specific tools inside ManageJenkins->Tools->Check JDK and Maven3.
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = "ap-south-1"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/bhavanishankar26/Shopping-Cart.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn clean compile"
            }
           // Plug Ins -- Eclipse Temurin installer Plugin , openJDK-native-plugin ,Oracle Java SE Development Kit Installer
        }
        
        stage('Sonarqube Analysis') {
            steps {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.url=http://3.109.1.198:9000/ -Dsonar.login=squ_0b16f660e5bc4514a3d8da45c5b9b896a6931f15 -Dsonar.projectName=shopping-cart \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=shopping-cart '''
                // Plugins installed  -   SonarQube Scanner for Jenkins,
                // Defined inside ManageJenkins->Tools->sonar-scanner
                // Public IP address will be changed after restarting the Jenkins Server .
            }
        }
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                // Added Plugin -- OWASAP inside Jekins 
            }
        }
        
        stage('Build App') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        stage('Deploy to JFrog Artifactory') {
            environment {
                // Define the target repository in Artifactory
                TARGET_REPO = 'my-local-repo'

                  // Plugin installed -  JFrog , Artifactory , Artifact Repository Parameter Plugin
            }
            
            steps {
                script {
                    try {
                        def server = Artifactory.newServer url: 'http://3.109.1.198:8081/artifactory/r123-1/', credentialsId: 'jfrog-cred'
                        def uploadSpec = """{
                            "files": [
                                {
                                    "pattern": "target/*.jar",
                                    "target": "${TARGET_REPO}/"
                                }
                            ]
                        }"""
                        
                        server.upload(uploadSpec)
                    } catch (Exception e) {
                        error("Failed to deploy artifacts to Artifactory: ${e.message}")
                    }
                }
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker', toolName: 'docker') {
                    sh "docker build -t shopping-cart:latest -f docker/Dockerfile ."
                    sh "docker tag shopping-cart:latest gadebhavani26/shopping-cart:latest"
                    sh "docker push gadebhavani26/shopping-cart:latest"
                    }
                    // Plug in -CloudBees Docker Build and Publish plugin 1.4.0
                    // Docker Version1.4
                    // Docker Pipeline
                    //docker-build-step
                }
            }
        }
        // stage("Deploy to EKS") {
        //     steps {
        //         script {
        //             dir('.') {
        //                 sh "aws eks --region ap-south-1 update-kubeconfig --name terraform-eks-demo"
        //                 sh "kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.0/deploy/static/provider/cloud/deploy.yaml"
        //                 sh "kubectl apply -f deploymentservice.yml"
        //             }
        //         }
        //     }
        // }
        stage('Trigger CD pipeline'){
           steps{
               build job:"CD",wait: true

               // K8s Ingress service will be used for delivering the application inside CD pipeline
           }
        }
    }
}
