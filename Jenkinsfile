pipeline {
agent{
    kubernetes{
        label 'slave'
    }
}
    environment {
        ZONE = "europe-west4-a"
        PROJECT_ID = "dev-infra-378919"
    }

    stages{
        
        stage("git checkout"){
            steps{
                script{
        git(
            url: 'https://github.com/eugenerodolph/jenkins-go-app.git',
            credentialsId: 'git-token',
            branch: 'main'
            )
                }
            }
        }
        
        stage("Building Application Docker Image"){
            steps{
                script{
                    sh 'gcloud auth configure-docker eu.gcr.io'
                    sh 'docker build . -t eu.gcr.io/dev-infra-378919/jenkins/hello-app:${BUILD_NUMBER}'
                    }
                }
            }
        stage("Pushing Application Docker Image to Google Artifact Registry"){
            steps{
                script{
                    sh 'docker push eu.gcr.io/dev-infra-378919/jenkins/hello-app:${BUILD_NUMBER}'
        }}}
        stage ("Updating Deployment Manifest") {
            steps {
                script {
                    sh 'sed -i s+eu.gcr.io/dev-infra-378919/jenkins/hello-app:v1+eu.gcr.io/dev-infra-378919/jenkins/hello-app:${BUILD_NUMBER}+g manifests/deployment.yaml'
                }
            }
        }
        stage("Application Deployment on Google Kubernetes Engine"){
            steps{
                script{
                    sh "gcloud container clusters get-credentials app-cluster --zone ${env.ZONE} --project ${env.PROJECT_ID}"
                    sh 'kubectl apply -f manifests/.'
                }
            }
        }
    }
    }
