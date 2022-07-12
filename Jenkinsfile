pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube'
                    }

                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
 
                }  
            }
        }
        stage("docker build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_password', variable: 'nexus_passwordsnipp')]) {
                             sh '''
                                docker build -t 34.82.248.16:8083/springapp:${VERSION} .
                                docker login -u admin -p $nexus_passwordsnipp 34.82.248.16:8083
                                docker push 34.82.248.16:8083/springapp:${VERSION}
                                docker rmi 34.82.248.16:8083/springapp:${VERSION}
                            '''
                    }
                }
            }
        }
         // stage("identify misconfigs using datree in helm"){
         //     steps{
         //         script{
	 //             dir('kubernetes/') {
        //                 withEnv(['DATREE_TOKEN=4156e936-1eb6-4b4e-982e-7dcd4e288820']) {
       // sh 'helm datree test myapp/'
                        //}
//}
//                 }
//             }
//         }
        stage("helm push repo stage"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_password', variable: 'nexus_passwordsnipp')]) {
                          dir('kubernetes/') {
                             sh '''
                                 helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                 tar -czvf  myapp-${helmversion}.tgz myapp/
                                 curl -u admin:$nexus_password http://34.82.248.16:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                          }
                    }
                }
            }
        }
        stage('manual approval'){
            steps{
                script{
                    timeout(10) {
                        mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> Go to build url and approve the deployment request <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "thiruece.r@gmail.com";
                        input(id: "Deploy Gate", message: "Deploy ${params.project_name}?", ok: 'Deploy')
    // some block
                    }
                }
            }
        }
        stage('Deploying application on k8s cluster') {
            steps {
               script{
                   withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
                        dir('kubernetes/') {
                          sh 'helm upgrade --install --set image.repository="34.82.248.16:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ '
                          sh 'kubectl scale myjavaapp-myapp --replicas=5'
                        }
                    }
               }
            }
        }

        stage('verifying app deployment'){
            steps{
                script{
                     withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
                         sh 'kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl myjavaapp-myapp:8080'

                     }
                }
            }
        }
    }

    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "thiruece.r@gmail.com";  
		 }
	   }
}