pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' 
            }
        }
      stage('Unit Tests - JUnit and Jacoco') {
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
                pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
              }
            }
        }
      stage('SonarQube - SAST') {
            steps {
              sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecopstraining.eastus.cloudapp.azure.com:9000 -Dsonar.login=sqp_c6e4621187b48d162feb9c96e947dcf2484fccf6"
            }
        }

      stage('Docker Build and Push') {
	            steps {
                withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
                        sh 'printenv'
                        sh 'sudo docker build -t kelvinchowinfocepts/numeric-app:""$GIT_COMMIT"" .'
                        sh 'docker push kelvinchowinfocepts/numeric-app:""$GIT_COMMIT""'
                        }
                    }
            } //comment
      stage('Kubernetes Deployment -Dev') {
	            steps {
                withKubeConfig([credentialsId: "kubeconfig"]) {
                        sh "sed -i 's#replace#siddharth67/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                        sh "kubectl apply -f k8s_deployment_service.yaml"
                        }
                    }
            }      
    }
}