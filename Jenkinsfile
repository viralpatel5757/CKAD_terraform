def projectId = "methodical-aura-308217"

pipeline {
    environment {
        registry = "viralpatel5757/external-new-image"
        registryCredential = 'viralpatel5757'
        app = ''
    }
    
   agent any

   stages {
    //   stage('stage 1 : SCM') {
    //         steps {
    //             git url: 'https://github.com/viralpatel5757/deloitted_external.git'
    //         }
    //     }
        stage('stage-1 :Cloning our Git') {
          steps {
              echo '****************************** Stage 1'
              dir("${env.WORKSPACE}/external") {
               git 'https://github.com/viralpatel5757/deloitted_external.git'
                }
            }
        }
    
      stage("stage 2 : SonarQube Analysis") {
          
          steps {
             script {
                   def scannerHome = tool 'SonarQube Scanner 2.13';
                   withSonarQubeEnv("sonarqube") {
                   sh "${scannerHome}/bin/sonar-scanner"
                   }
                }
            }
        }
        
        

        stage('stage-3 :Initialize') {
          steps{
              echo '****************************** Stage 2'
              dir("${env.WORKSPACE}/external") {
                    script {
                        def dockerHome = tool 'myDocker'
                        env.PATH = "${dockerHome}/bin:${env.PATH}"
                    }    
                }
            }    
        }

        stage('stage-4 :Building our image') {
          steps{
              echo '****************************** Stage 3'
              dir("${env.WORKSPACE}/external") {
                 script {
                      app = docker.build registry + ":$BUILD_NUMBER"
                    //   sh "sudo -S docker build -t viralpatel5757/external-new-image:$BUILD_NUMBER ."
                    }   
                }
            }
        }

        stage('stage-5 :Deploy our image') {
            steps{
                echo '****************************** Stage 4'
                script {
                    docker.withRegistry( 'https://registry.hub.docker.com', 'docker-hub-credentials' ) {
                    app.push("${env.BUILD_NUMBER}")                    }
                }
            }
        }

        stage('Stage 6 - workspace and versions') {
            steps {
                sh 'echo $WORKSPACE'
                //sh 'docker --version'
                sh 'gcloud version'
                sh 'nodejs -v'
                sh 'npm -v'
            }
        }

        stage('Stage 7 - build external') {
            // setting env variable so external default port does not conflict with jenkins
            environment {
                PORT = 8081
            }
            steps {
                dir("${env.WORKSPACE}/external"){
                    echo 'Retrieving source from github' 
                    git branch: 'master',
                        url: 'https://github.com/viralpatel5757/deloitted_external.git'
                    echo 'Did we get the source?' 
                    sh 'ls -a'
                    echo 'install dependencies' 
                    sh 'npm install'
                    echo 'Run tests'
                    sh 'npm test'
                    echo 'Tests passed on to build Docker container'
                    echo "build id = ${env.BUILD_ID}"
                    sh "gcloud builds submit -t gcr.io/${projectId}/external-new-image:v1.${env.BUILD_ID} ."
                }
            }
        }
        stage('Stage 8 - build internal') {
            steps {
                dir("${env.WORKSPACE}/internal"){
                  echo 'Retrieving source from github' 
                    git branch: 'master',
                        url: 'https://github.com/viralpatel5757/deloitted_internal.git'
                    sh "ls -a"
                    echo 'install dependencies' 
                    sh 'npm install'
                    echo 'Run tests'
                    sh 'npm test'
                    echo 'Tests passed on to build Docker container'
                    echo "build id = ${env.BUILD_ID}"
                    sh "gcloud builds submit -t gcr.io/${projectId}/internal-new-image:v1.${env.BUILD_ID} ."
                }
            }
        }
        // stage('Stage 6 - deploy using terraform') {
        //     steps {
        //         dir("${env.WORKSPACE}/terraform"){
        //           echo 'Retrieving source from github' 
        //             git branch: 'main',
        //                 url: 'https://github.com/viralpatel5757/terraform.git'
        //             sh "ls -a"  
        //             sh 'terraform init'
        //              sh "terraform apply -var project_id=${projectId} -var external_image_name=external-new-image:v1.${env.BUILD_ID}  -var internal_image_name=internal-new-image:v1.${env.BUILD_ID}  -auto-approve"
        //           }
        //     }
            
        // }

        stage('Stage 9') {
            steps {
                echo 'Get cluster credentials'
                sh 'gcloud container clusters get-credentials deloitted-events-feed-cluster --zone us-central1-a --project methodical-aura-308217'
                
                echo 'Update the internal image and deployment'
                echo "gcr.io/methodical-aura-308217/internal-new-image:1.${env.BUILD_ID}"
                sh "kubectl set image deployment/deloitted-internal-events-feed-deployment deloitted-internal-events-feed=gcr.io/methodical-aura-308217/internal-new-image:v1.${env.BUILD_ID} -n=deloitted-events-feed --record"
                
                echo 'Update the external image and deployment'
                echo "gcr.io/methodical-aura-308217/external-new-image:1.${env.BUILD_ID}"
                sh "kubectl set image deployment/deloitted-external-events-feed-deployment deloitted-external-events-feed=gcr.io/methodical-aura-308217/external-new-image:v1.${env.BUILD_ID} -n=deloitted-events-feed --record"
            }
        }

   }
}
    
    
