def registry  ='https://trial1vp1km.jfrog.io/'
pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
         }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main' , url: 'https://github.com/Karthik351/End-to-End-Pipeline.git'
            }
        }
         stage('Versioning') {
    steps {
        script {
            sh 'mvn versions:set -DnewVersion=1.0.${BUILD_NUMBER}'
        }
    }
}
        stage('Maven Compile') {
            steps {
               echo 'Maven Compile Started'
               sh 'mvn compile'
            }
        } 
         stage('Maven Test') {
            steps {
               echo 'Maven Test Started'
               sh 'mvn test'
            }
        } 
        stage('File System Scan by Trivy') {
            steps {
               echo 'Trivy Scan Started'
               sh 'trivy fs --format table --output trivy-fs-output.txt .'
            }
        } 
        stage('Sonar Analysis') {
            steps {
               withSonarQubeEnv('sonar-scanner') {
                sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=SpringBootApp -Dsonar.projectKey=SpringBoot \
                                                       -Dsonar.java.binaries=. -Dsonar.exclusions=**/trivy-fs-output.txt '''
               }
            }
        } 
        stage('Quality Gate') {
            steps {
              timeout(time: 1, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true, credentialsId: 'sonar-scanner'  
          }
        } 
      }
       stage('Maven Package') {
            steps {
               echo 'Maven package Started'
               sh 'mvn package'
          }
        } 
        stage("Jar Publish") {
            steps {
                script {
                        echo '<--------------- Jar Publish Started --------------->'
                         def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrog"
                         def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                         def uploadSpec = """{
                              "files": [
                                {
                                  "pattern": "target/springbootApp.jar",
                                  "target": "end-end-pipeline-libs-release",
                                  "flat": "false",
                                  "props" : "${properties}",
                                  "exclusions": [ "*.sha1", "*.md5"]
                                }
                             ]
                         }"""
                         def buildInfo = server.upload(uploadSpec)
                         buildInfo.env.collect()
                         server.publishBuildInfo(buildInfo)
                         echo '<--------------- Jar Publish Ended --------------->'  
                
                }
            }   
        } 
        stage('Build Docker Image and TAG') {
            steps {
                script {
                    // Build the Docker image using the renamed JAR file
                    script {
                            sh 'docker build -t springbootapp:latest .'
                        }
                }   
            }
        }
//         stage('Docker Image Scan') {
//             steps {
//                 sh 'trivy image --format table --scanners vuln -o trivy-image-report.html springbootapp:latest'
//             }
//         }
//         stage('Push Docker Image') {
//             steps {
//                 script {
//                         sh "docker image tag springbootapp:latest bkrrajmali/springbootapp:latest"
//                         sh "docker push bkrrajmali/springbootapp:latest"

//                     }
//                 }
//             }

//             stage ('Push Docker Image to AWS ECR') {
//         steps {
//             script {
//                 sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 058264323019.dkr.ecr.us-east-1.amazonaws.com'
//                 sh 'docker tag springbootapp:latest 058264323019.dkr.ecr.us-east-1.amazonaws.com/myrepo:latest'
//                 sh 'docker push 058264323019.dkr.ecr.us-east-1.amazonaws.com/myrepo:latest'
//             }
//         }
//     }
//     stage('Deploy To Kubernetes') {
//         steps {
//         withKubeConfig(caCertificate: '', clusterName: 'eks2', contextName: '', credentialsId: 'k8-cred', namespace: 'default', restrictKubeConfigAccess: false, serverUrl: 'https://A7C7808EC02839C9FBA67DFE109DBC33.gr7.us-east-1.eks.amazonaws.com')
//          {
//             sh "kubectl apply -f eks-deploy-k8s.yaml"
//             }
//         }
//     }
    }
  }
