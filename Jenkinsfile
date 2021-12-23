/*
LFS261
Jenkisnsfile Global del proyecto
Creo un unico pipeline para las 3 apps

*/
pipeline {
    agent none 

    stages {                                    // Defino las etapas
        stage('build-worker') {
            agent{
                docker {
                    image "maven:3.6.1-jdk-8-alpine"
                    args '-v $HOME/.m2:/root/.m2'   // Para que no descargue de internet la imagen cada vez que la usemos
                }
            }
            when{                               // Ejecuto en cualquier branch, pero
                changeset '**/worker/**'        // Solo se ejecuta si hago cambios en la carpeta worker
            }
            steps {                             // primera etapa, compilar
                echo 'Compiling worker file'
                dir('worker'){                  // Cambio al directorio worker
                    sh 'mvn compile'            // Comando compilar de maven desde SH
                }
            }
        }

        stage('build-vote') {
            agent {
                docker{
                    image 'python:2.7.16-slim'
                    args '-u root' // Permite que pip instale paquetes dentro del contenedor
                }
            }
            when{                               // Ejecuto en cualquier branch, pero
                changeset '**/vote/**'        // Solo se ejecuta si hago cambios en la carpeta vote
            }
            steps {                             // primera etapa, compilar
                echo 'Building vote App...'
                dir('vote'){                  // Cambio al directorio result
                    sh 'pip install -r requirements.txt'            // instalo los requerimientos
                }
            }
        }

        stage('build-result') {
            agent {
                docker{
                    image 'node:8.16-alpine'
                }
            }
            when{                               // Ejecuto en cualquier branch, pero
                changeset '**/result/**'        // Solo se ejecuta si hago cambios en la carpeta result
            }
            steps {                             // primera etapa, compilar
                echo 'Compiling result App...'
                dir('result'){                  // Cambio al directorio result
                    sh 'npm install'            // instalo nodejs
                }
            }
        }
        
        stage('test-worker') {                         // Segunda etapa, test
            agent{
                docker {
                    image "maven:3.6.1-jdk-8-alpine"
                    args '-v $HOME/.m2:/root/.m2'   // Para que no descargue de internet la imagen cada vez que la usemos
                }
            }
            when{
                changeset '**/worker/**'        // Solo se ejecuta si hago cambios en la carpeta worker
            }
            steps {
                echo 'Running Unit Tests on Worker App'
                dir('worker'){
                    sh 'mvn clean test '
                }
            }
        }

        stage('test-vote') {                         // Segunda etapa, test
            agent {
                docker{
                    image 'python:2.7.16-slim'
                    args '-u root' // Permite que pip instale paquetes dentro del contenedor
                }
            }
            when{
                changeset '**/vote/**'        // Solo se ejecuta si hago cambios en la carpeta result
            }
            steps {
                echo 'Running Nose Tests on vote App...'
                dir('vote'){
                    sh 'pip install -r requirements.txt'            // instalo los requerimientos
                    sh 'nosetests -v'
                }
            }
        }

        stage('test-result') {                         // Segunda etapa, test
            agent {
                docker{
                    image 'node:8.16-alpine'
                }
            }            
            when{
                changeset '**/result/**'        // Solo se ejecuta si hago cambios en la carpeta result
            }
            steps {
                echo 'Running Unit Tests on result App...'
                dir('result'){
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }
        
        stage('package-worker') {                      // tercera etapa, package, genera los archivos .jar
            agent{
                docker {
                    image "maven:3.6.1-jdk-8-alpine"
                    args '-v $HOME/.m2:/root/.m2'   // Para que no descargue de internet la imagen cada vez que la usemos
                }
            }
            when{
                changeset '**/worker/**'        // Solo se ejecuta si hago cambios en la carpeta worker
            }
            steps {
                echo 'Packaging Worker App'
                dir('worker'){
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }
        
        stage('docker-image-worker'){
            agent any // PORQUE QUIERO EJECUTAR DIRECTAMENTE ESTO EN EL HOST, NO dentro del contenedor que compila
            when{
                changeset '**/worker/**'        // Solo se ejecuta si hago cambios en la carpeta worker
                branch 'feature/monopipe'                 // Solo se genera la imagen para el branch master
            }
            steps{
              echo 'Packaging worker app with docker'
              script{
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

        stage('docker-image-vote'){
            agent any 
            when{
                changeset '**/vote/**'  
                branch 'feature/monopipe'         
            }
            steps{
              echo 'Packaging vote app with docker'
              script{
                docker.withRegistry('https://index.docker.io/v1/', 'dockerhublogin') {
                    def workerImage = docker.build("lrbono/vote:v${env.BUILD_ID}", "./vote")
                    workerImage.push()
                    workerImage.push("${env.BRANCH_NAME}")
                }
              }
            }
        }

        stage('docker-image-result'){
            agent any 
            when{
                changeset '**/result/**'  
                branch 'feature/monopipe'         
            }
            steps{
              echo 'Packaging result app with docker'
              script{
                docker.withRegistry('https://index.docker.io/v1/', 'dockerhublogin') {
                    def workerImage = docker.build("lrbono/result:v${env.BUILD_ID}", "./result")
                    workerImage.push()
                    workerImage.push("${env.BRANCH_NAME}")
                }
              }
            }
        }

        // Este stage hace el deploy de todas las partes de la app usando el docker-compose
        stage('deploy to dev'){
            agent any
            when{
                branch 'feature/monopipe'
            }
            steps{
                echo 'Deploy instavote app with docker compose'
                sh 'docker-compose up -d'
            }
        } 
    }
    
    post{                                       // Post implementación
        always{                                 // Indica ejecuatar siempre
            echo 'Pipeline for Instavote App is complete!'
        }
    }
}

