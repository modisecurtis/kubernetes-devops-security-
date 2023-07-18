pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //testing
            }
        }

        stage('Unit Tests') {
              steps {
                sh "mvn test"
              }
              post {
                always {
                  junit 'target/surefire-reports/*.xml'
                  jacoco execPattern: 'target/jacoco.exec'
                }
              }
        }
// docker hub
        // stage('Docker Build and Push') {
        //     steps {
        //       withDockerRegistry([credentialsId: "docker-hub", url:""]) {
        //       sh 'printenv'
        //       sh 'docker build -t curtissananapo/numeric-app:""$GIT_COMMIT"" .'
        //       sh 'docker push curtissananapo/numeric-app:""$GIT_COMMIT""'
        //       }
        //     }
        // }

        stage('Push image') {
        /* Finally, we'll push the image with two tags:
        * First, the incremental build number from Jenkins
        * Second, the 'latest' tag. */
        withCredentials([usernamePassword( credentialsId: 'docker-hub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {

        docker.withRegistry('', 'docker-hub-credentials') {
        sh "docker login -u ${USERNAME} -p ${PASSWORD}"
        myImage.push("${env.BUILD_NUMBER}")
        myImage.push("latest")
        }

//         stage('Kubernetes Deployment - LOCAL') {
//             steps {
//               withKubeConfig([credentialsId: 'kubeconfig']) {
//               sh "sed -i 's#replace#curtissananapo/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
//               sh "kubectl apply -f k8s_deployment_service.yaml"
//               }
//             }
//         }
    }
}
