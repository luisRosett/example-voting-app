pipeline {
    agent none

    stages {
        stage('worker build') {
            agent {
                docker {
                    image 'maven:alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                changeset "**/worker/**"
            }
            steps {
                echo 'Compiling worker app'
                dir('worker'){
                        sh 'mvn compile'
                }
            }
        }
        stage('worker test') {
            agent {
                docker {
                    image 'maven:alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                changeset "**/worker/**"
            }
            steps {
                echo 'Running Unit Tests on worker app'
                dir('worker'){
                    sh 'mvn clean test'
                }
            }
        }
        stage('worker package') {
            agent {
                docker {
                    image 'maven:alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                changeset "**/worker/**"
            }
            steps {
                echo 'Packaging worker app'
                dir('worker'){
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }
        stage('worker docker-package') {
            agent any
            when {
                changeset "**/worker/**"
                branch 'master'
            }
            steps {
                echo 'Packaging worker app with docker'
                script {
                     docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {

                        def workerImage = docker.build("luisroset/worker:v${env.BUILD_ID}", "./worker")

                        /* Push the container to the custom Registry */
                        
                        workerImage.push("${env.BUILD_NUMBER}")
                        workerImage.push("latest") 
                    }
                }
            }
        }
        stage('vote build') {
            agent {
                docker {
                    image 'python:latest'
                    args '--user root'
                }
            }
            when {
                changeset "**/vote/**"
            }
            steps {
                echo 'Compiling python app'
                dir('vote'){
                        sh 'pip install -r requirements.txt'
                }
            }
        }
        stage('vote test') {
            agent {
                docker {
                    image 'python:latest'
                    args '--user root'
                }
            }
            when {
                changeset "**/vote/**"
            }
            steps {
                echo 'Running Unit Tests on python app'
                dir('vote'){
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }
        stage('vote integration') {
            agent any
            when {
                changeset "**/vote/**"
                branch 'master'
            }
            steps {
                echo 'Running Integration Tests on vote app'
                dir('vote'){
                    sh 'integration_test.sh'
                }
            }
        }
         
        stage('vote docker-package') {
            agent any
            when {
                changeset "**/vote/**"
                branch 'master'
            }
            steps {
                echo 'Packaging vote app with docker'
                script {
                     docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {

                        def workerImage = docker.build("luisroset/vote:v${env.BUILD_ID}", "./vote")

                        /* Push the container to the custom Registry */
                        
                        workerImage.push("${env.BUILD_NUMBER}")
                        workerImage.push("latest") 
                    }
                }
            }
        }
        stage('result build') {
            agent {
                docker {
                    image 'node:alpine'
                }
            }
            when {
                changeset "**/result/**"
            }
            steps {
                echo 'Compiling result app'
                dir('result'){
                        sh 'npm install'
                }
            }
        }
        stage('result test') {
            agent {
                docker {
                    image 'node:alpine'
                }
            }
            when {
                changeset "**/result/**"
            }
            steps {
                echo 'Running Unit Tests on python app'
                dir('result'){
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }
        stage('result docker-package') {
            agent any
            when {
                changeset "**/result/**"
                branch 'master'
            }
            steps {
                echo 'Packaging result app with docker'
                script {
                     docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {

                        def Image = docker.build("luisroset/:v${env.BUILD_ID}", "./")

                        /* Push the container to the custom Registry */
                        
                        Image.push("${env.BUILD_NUMBER}")
                        Image.push("latest") 
                    }
                }
            }
        }
        stage('Sonarqube') {
          agent any
          /*when{
            branch 'fea'
          }*/
          tools {
            jdk "JDK11" // the name you have given the JDK installation in Global Tool Configuration
          }

          environment{
            sonarpath = tool 'SonarScanner'
          }

          steps {
                echo 'Running Sonarqube Analysis..'
                withSonarQubeEnv('sonar-instavote') {
                  sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties -Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400"
                }
          }
        }


        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
                
            }
        }
    } 
    post {
        always{
            echo 'Build pipeline for instavote app is complete...'

        }
        failure{
            slackSend (channel: "instavote-cd",message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        success{
            slackSend (channel: "instavote-cd",message: "Build Succeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
    }
}
