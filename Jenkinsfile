pipeline{
    agent any

stages {
    stage('SCM Checkout') {
        steps {
            git 'https://github.com/damodaranj/my-app.git'
        }
    }
    
    stage('maven-buildstage') {
        steps {
            script {
                def mvnHome = tool name: 'maven3', type: 'maven'
                sh "${mvnHome}/bin/mvn clean package"
                sh 'mv target/myweb*.war target/newapp.war'
            }
        }
    }
    
    stage('SonarQube Analysis') {
        steps {
            script {
                def mvnHome = tool name: 'maven3', type: 'maven'
                withSonarQubeEnv('sonar') {
                    sh "${mvnHome}/bin/mvn sonar:sonar"
                }
            }
        }
    }
    
    stage('Build Docker Image') {
        steps {
            sh 'docker build -t saidamo/myweb:0.0.2 .'
        }
    }
    
    stage('Docker Image Push') {
        steps {
            withCredentials([string(credentialsId: 'dockerPass', variable: 'dockerPassword')]) {
                sh "docker login -u saidamo -p ${dockerPassword}"
            }
            sh 'docker push saidamo/myweb:0.0.2'
        }
    }
    
    stage('Nexus Image Push') {
        steps {
            sh "docker login -u admin -p admin123 13.233.74.99:8083"
            sh "docker tag saidamo/myweb:0.0.2 13.233.74.99:8083/damo:1.0.0"
            sh 'docker push 13.233.74.99:8083/damo:1.0.0'
        }
    }

    stage('Remove Previous Container') {
        steps {
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                sh 'docker rm -f tomcattest'
            }
        }
    }
    
    stage('Docker deployment') {
        steps {
            sh 'docker run -d -p 8090:8080 --name tomcattest saidamo/myweb:0.0.2'
        }
    }
}

