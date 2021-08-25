pipeline {
  agent none
  stages {
    stage('Worker Build') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        changeset '**/worker/**'
      }
      steps {
        echo 'Compiling worker app..'
        dir(path: 'worker') {
          sh 'mvn compile'
        }

      }
    }

    stage('Worker Test') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        changeset '**/worker/**'
      }
      steps {
        echo 'Running Unit Tests on worker app.'
        dir(path: 'worker') {
          sh 'mvn clean test'
        }

      }
    }

    stage('Worker Package') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        branch 'master'
        changeset '**/worker/**'
      }
      steps {
        echo 'Packaging worker app'
        dir(path: 'worker') {
          sh 'mvn package -DskipTests'
          archiveArtifacts(artifacts: '**/target/*.jar', fingerprint: true)
        }

      }
    }

    stage('Worker docker-package') {
      agent any
      when {
        changeset '**/worker/**'
        branch 'master'
      }
      steps {
        echo 'Packaging worker app with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
            def workerImage = docker.build("chanzin/worker:v${env.BUILD_ID}", "./worker")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
            workerImage.push("latest")
          }
        }

      }
    }

    stage('Result Build') {
      agent {
        docker {
          image 'node:16.6.1-alpine'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'Compiling result app..'
        dir(path: 'result') {
          sh 'npm install'
        }

      }
    }

    stage('Result Test') {
      agent {
        docker {
          image 'node:16.6.1-alpine'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'Running Unit Tests on result app.'
        dir(path: 'result') {
          sh 'npm install'
          sh 'npm test'
        }

      }
    }

    stage('Result docker-package') {
      agent any
      when {
        changeset '**/result/**'
        branch 'master'
      }
      steps {
        echo 'Packaging Result App with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
            def workerImage = docker.build("chanzin/result:v${env.BUILD_ID}", "./result")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
            workerImage.push("latest")
          }
        }

      }
    }

    stage('Vote Build') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Compiling vote app..'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
        }

      }
    }

    stage('Vote Test') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Running Unit Tests on vote app.'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
          sh 'nosetests -v'
        }

      }
    }

    stage('Vote docker-package') {
      agent any
      when {
        changeset '**/vote/**'
        branch 'master'
      }
      steps {
        echo 'Packaging Vote app with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
            def workerImage = docker.build("chanzin/vote:v${env.BUILD_ID}", "./vote")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
            workerImage.push("latest")
          }
        }

      }
    }
      
    stage('Sonarqube') {
      agent any
/*      when{
        branch 'master'
      }
 */     
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

    stage('Deploy to Dev') {
      agent any
      when{
          branch 'master'
      }
      steps {
        sh 'docker-compose up -d'
      }
    }

  }
  post {
    always {
      echo 'Pipeline for InstaVote App  is completed...'
    }

    failure {
      echo 'Build Failed'
    }

    success {
      echo 'Build Succeed'
    }

  }
}