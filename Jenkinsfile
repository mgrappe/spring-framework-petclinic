pipeline {
    agent any
    tools {
        maven "maven 3.6"
    }
     environment {
        NEXUS_ARTIFACT_VERSION= "${env.BUILD_NUMBER}"
    }

    options {
        parallelsAlwaysFailFast()
    }
    stages {
        stage('Non-Parallel Stage') {
            steps {
                echo 'This stage will be executed first.'
            }
        }
        stage('Parallel Stage') {
            parallel {
                   stage('Checkstyle') {
                        steps{
                            // Run the maven build with checkstyle
                            sh "mvn clean package checkstyle:checkstyle"
                         }
                     }
                    stage('Sonarqube') {
                        steps {
                           // withSonarQubeEnv('SonarQube') {
                           // sh "mvn  clean package sonar:sonar -Dsonar.host_url=$SONAR_HOST_URL "
                           // }
                           echo 'Sonarqube'
                         }
                    }
            }
        }
        stage('Publish in Nexus') {
            steps {
                nexusPublisher nexusInstanceId: 'Nexus',
                nexusRepositoryId: 'releases',
                packages: [[$class: 'MavenPackage',
                mavenAssetList: [[classifier: '', extension: '', filePath: 'target/petclinic.war']], mavenCoordinate: [artifactId: 'spring-framework-petclinic', groupId: 'org.springframework.samples', packaging: 'war', version: NEXUS_ARTIFACT_VERSION]]]
            }
        }
        stage('Build image') {
            steps{
                script{
                    def customImage = docker.build("petclinic-project")
                }
            }
        }
        stage('Parallel Stage Prod') {
        parallel{
        stage ('Run Test Image'){
            steps{
                sh "docker stop petclinic-test && docker rm petclinic-test || true"
                sh 'docker run -d --name petclinic-test -p 8090:8080 petclinic-project'
            }
        }
        stage ('Run uat image'){
            steps{
                sh "docker stop petclinic-uat && docker rm petclinic-uat || true"
                sh 'docker run -d --name petclinic-uat -p 8190:8080 petclinic-project'
            }
        }

        }
        }



    }
}
