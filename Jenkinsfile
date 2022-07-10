pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            agent {
                docker{
                    image 'openjdk:11'
                }
            }
            steps{
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube'
                    }

                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "pipeline aborted due to quality gate failur: ${qg.status}"
                        }
                    }
                }
            }
        }
        stage(docker build & docker push){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_password', variable: 'nexus_passwordsnipp')]) {
    // some block
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
    }
}



