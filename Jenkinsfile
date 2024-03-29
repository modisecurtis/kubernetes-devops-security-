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
        stage('Mutation Tests - PIT') {
            steps {
              sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
            post {
              always {
                pitmutation mutationStatsFile: '**target/pit-reports/**/mutations.xml'
              }
            }
          }

// SAST
        // stage('SonarQube - SAST') {
        //     steps {
        //       sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://localhost:9000 -Dsonar.login=39d62e1c66a687cc0a40bb1b8fe092e83e370b48"
        //     }
        //   }

        stage('SonarQube - SAST') {
          steps {
            withSonarQubeEnv('SonarQube') {
              sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://craftbox:9000"
            }
            timeout(time: 2, unit: 'MINUTES') {
              script {
                waitForQualityGate abortPipeline: true
              }
            }
          }
        }

        stage('Vulnerability Scan - Docker ') {
          steps {
            sh "mvn dependency-check:check"
          }
          post {
            always {
              dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
            }
          }
        }

// docker hub-1
        stage('Docker Build and Push') {
            steps {
              withDockerRegistry([credentialsId: "docker-hub", url:""]) {
              sh 'printenv'
              sh 'docker build -t curtissananapo/numeric-app:""$GIT_COMMIT"" .'
              sh 'docker push curtissananapo/numeric-app:""$GIT_COMMIT""'
              }
            }
        }

//
        stage('Kubernetes Deployment - LOCAL') {
            steps {
              withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "sed -i 's#replace#curtissananapo/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
              sh "kubectl apply -f k8s_deployment_service.yaml"
              }
            }
        }
    }
}
