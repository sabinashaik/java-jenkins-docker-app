pipeline {
   agent any

    environment {
       AWS_REGION = 'us-east-1'
       AWS_ACCOUNT_ID = '595658222114'
       ECR_REPO = 'java-jenkins-docker-app'
       EKS_CLUSTER = 'java-eks-cluster'
       IMAGE_TAG = "${env.BUILD_NUMBER}"
       ECR_URI = "${595658222114}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
       IMAGE_URI = "${ECR_URI}:${IMAGE_TAG}"
   }

 stages {

 

       stage('Clean Workspace') {
           steps {
               cleanWs()
           }
       }

 

       stage('Clone') {
           steps {
               git(
                   url: 'git@github.com:sabinashaik/java-jenkins-docker-app.git',
                   branch: 'main',
                   credentialsId: 'git'
               )
           }
       }

 

       stage('SonarQube Analysis') {
           steps {
               withSonarQubeEnv('sonar-server') {
                   sh '''
                   docker run --rm \
                     -v "$PWD":/usr/src \
                     sonarsource/sonar-scanner-cli \
                     -Dsonar.projectKey=java-jenkins-docker-app \
                     -Dsonar.sources=/usr/src/src \
                     -Dsonar.host.url=$SONAR_HOST_URL \
                     -Dsonar.login=$SONAR_AUTH_TOKEN
                   '''
               }
           }
       }

 

       stage('Quality Gate') {
           steps {
               timeout(time: 5, unit: 'MINUTES') {
                   waitForQualityGate abortPipeline: true
               }
           }
       }

 

       stage('Docker Build') {
           steps {
               sh '''
               docker build -t ${ECR_REPO}:${IMAGE_TAG} .
               docker tag ${ECR_REPO}:${IMAGE_TAG} ${IMAGE_URI}
               '''
           }
       }

 

       stage('ECR Login') {
           steps {
               withCredentials([
                   string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                   string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
               ]) {
                   sh '''
                   aws ecr get-login-password --region ${AWS_REGION} | \
                   docker login --username AWS --password-stdin \
                   ${595658222114}.dkr.ecr.${AWS_REGION}.amazonaws.com
                   '''
               }
           }
       }

 

       stage('Push Image to ECR') {
           steps {
               sh '''
               docker push ${IMAGE_URI}
               '''
           }
       }

 

       stage('ECR Image Scan Check') {
           steps {
               sh '''
               echo "Waiting for ECR image scan..."
               sleep 40

 

               aws ecr describe-image-scan-findings \
                 --repository-name ${ECR_REPO} \
                 --image-id imageTag=${IMAGE_TAG} \
                 --region ${AWS_REGION} \
                 --query 'imageScanFindings.findingSeverityCounts'
               '''
           }
       }

 

       stage('Configure kubectl for EKS') {
           steps {
               sh '''
               aws eks update-kubeconfig \
                 --region ${AWS_REGION} \
                 --name ${EKS_CLUSTER}
               '''
           }
       }

 

       stage('Deploy to EKS') {
           steps {
               sh '''
               sed "s|IMAGE_PLACEHOLDER|${IMAGE_URI}|g" k8s/deployment.yaml > k8s/deployment-final.yaml

 

               kubectl apply -f k8s/deployment-final.yaml
               kubectl apply -f k8s/service.yaml
               '''
           }
       }

 

       stage('Rollout Status') {
           steps {
               sh '''
               kubectl rollout status deployment/java-jenkins-docker-app --timeout=120s
               kubectl get pods
               kubectl get svc
               '''
           }
       }
   }

 

   post {
       success {
           emailext(
               subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
               body: "Build successful. Image deployed: ${IMAGE_URI}",
               to: "sabinashaik228@gmail.com"
           )
       }

 

       failure {
           emailext(
               subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
               body: "Build failed. Check Jenkins console logs: ${env.BUILD_URL}",
               to: "sabinashaik228@gmail.com"
           )
       }

 

       always {
           cleanWs()
       }
 }
}
