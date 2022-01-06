pipeline {
  agent none
  stages {
    stage('build-worker') {
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
        echo 'Compiling worker file'
        dir(path: 'worker') {
          sh 'mvn compile'
        }

      }
    }

    stage('build-vote') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '-u root'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Building vote App...'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
        }

      }
    }

    stage('build-result') {
      agent {
        docker {
          image 'node:8.16-alpine'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'Compiling result App...'
        dir(path: 'result') {
          sh 'npm install'
        }

      }
    }

    stage('test-worker') {
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
        echo 'Running Unit Tests on Worker App'
        dir(path: 'worker') {
          sh 'mvn clean test '
        }

      }
    }

    stage('test-vote') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '-u root'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Running Nose Tests on vote App...'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
          sh 'nosetests -v'
        }

      }
    }

    stage('test-result') {
      agent {
        docker {
          image 'node:8.16-alpine'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'Running Unit Tests on result App...'
        dir(path: 'result') {
          sh 'npm install'
          sh 'npm test'
        }

      }
    }

    stage('package-worker') {
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
        echo 'Packaging Worker App'
        dir(path: 'worker') {
          sh 'mvn package -DskipTests'
          archiveArtifacts(artifacts: '**/target/*.jar', fingerprint: true)
        }

      }
    }

    stage('docker-image-worker') {
      agent any
      when {
        changeset '**/worker/**'
        branch 'feature/monopipe'
      }
      steps {
        echo 'Packaging worker app with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerhublogin') {
            // hago el Build con el Dockerfile
            def workerImage = docker.build("lrbono/worker:v${env.BUILD_NUMBER}", "./worker")
            workerImage.push()
            // Publico en Dockerhub
            workerImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('docker-image-vote') {
      agent any
      when {
        changeset '**/vote/**'
        branch 'feature/monopipe'
      }
      steps {
        echo 'Packaging vote app with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerhublogin') {
            def workerImage = docker.build("lrbono/vote:v${env.BUILD_ID}", "./vote")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('docker-image-result') {
      agent any
      when {
        changeset '**/result/**'
        branch 'feature/monopipe'
      }
      steps {
        echo 'Packaging result app with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerhublogin') {
            def workerImage = docker.build("lrbono/result:v${env.BUILD_ID}", "./result")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('deploy-to-dev-from-blueocean') {
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
      echo 'Pipeline for Instavote App is complete!'
    }

  }
}