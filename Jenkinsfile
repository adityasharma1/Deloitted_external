def projectId = "deloitte-demo-308622"

pipeline {
   agent any

   stages {
        stage('Stage 1 - workspace and versions') {
                        
            steps {
                echo '****************************** Stage 1'
                sh 'echo $WORKSPACE'
                //sh 'docker --version'
                sh 'gcloud version'
                sh 'nodejs -v'
                sh 'npm -v'
            }
        }
        stage('Stage 2 - get source and quality check') {
            // setting env variable so external default port does not conflict with jenkins
            environment {
                PORT = 8081
            }
            steps {
                echo '****************************** Stage 2'
                dir("${env.WORKSPACE}/external"){
                    echo 'Retrieving source from github' 
                    git branch: 'master',
                        url: 'https://github.com/adityasharma1/Deloitted_external.git'
                    echo 'Did we get the source?' 
                    sh 'ls -a'
                    echo 'install dependencies' 
                    sh 'npm install'
                    echo 'Run tests'
                    sh 'npm test'
                    
                }
                withSonarQubeEnv('sonarqube') {
                    script {
                        def scannerHome = tool 'sonarqube';
                        sh "${scannerHome}/bin/sonar-scanner \
                          -D sonar.login=admin \
                          -D sonar.password=admin \
                          -D sonar.projectKey=external \
                          -D sonar.host.url=http://35.193.67.2:9000/"
                    }
                }
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Stage 3 - build external') {
            steps {
                echo '****************************** Stage 3'
                dir("${env.WORKSPACE}/internal"){
                    echo "build id = ${env.BUILD_ID}"
                    sh "gcloud builds submit -t gcr.io/${projectId}/external-image:v2.${env.BUILD_ID} ."
                }
            }
        }
        
        stage('Stage 3 - deploy new image') {
            steps {
                echo 'Get cluster credentials'
                sh 'gcloud container clusters get-credentials demo-events-feed-cluster --zone us-central1-a --project deloitte-demo-308622'
                echo 'Update the image'
                echo "gcr.io/deloitte-demo-308622/external-image:2.${env.BUILD_ID}"
                sh "kubectl set image deployment/demo-external-events-feed-deployment demo-external-events-feed=gcr.io/deloitte-demo-308622/external-image:v2.${env.BUILD_ID} -n demo-events-feed --record"
            }
        }

   }
}