def projectId = "mar-roidtc321"

pipeline {
   agent any

   stages {
        stage('Stage 1 - workspace and versions') {
            steps {
                sh 'echo $WORKSPACE'
                //sh 'docker --version'
                sh 'gcloud version'
                sh 'nodejs -v'
                sh 'npm -v'
            }
        }
        stage('Stage 2 - build external') {
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
        // stage('Stage 3 - build internal') {
        //     steps {
        //         dir("${env.WORKSPACE}/internal"){
        //           echo 'Retrieving source from github' 
        //             git branch: 'main',
        //                 url: 'https://github.com/viralpatel5757/deloitted_internal.git'
        //             sh "ls -a"
        //             echo 'install dependencies' 
        //             sh 'npm install'
        //             echo 'Run tests'
        //             sh 'npm test'
        //             echo 'Tests passed on to build Docker container'
        //             echo "build id = ${env.BUILD_ID}"
        //             sh "gcloud builds submit -t gcr.io/${projectId}/internal-new-image:v1.${env.BUILD_ID} ."
        //         }
        //     }
        // }
        // stage('Stage 4 - deploy using terraform') {
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

        stage('Stage 5') {
            steps {
                echo 'Get cluster credentials'
                sh 'gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project mar-roidtc321'
                echo 'Update the image'
                echo "gcr.io/mar-roidtc321/internal-new-image:2.${env.BUILD_ID}"
                sh "kubectl set image deployment/events-external events-external=gcr.io/mar-roidtc321/internal-new-image:v2.${env.BUILD_ID} --record"
            }
        }

   }
}