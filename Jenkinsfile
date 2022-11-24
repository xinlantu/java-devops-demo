  //需要安装docker、docker pipeline插件

   pipeline {
       agent none
       stages {
           stage('Example Build') {
               agent {
                   docker 'maven:3-alpine'
                   //args 是指定    docker run 的所有指令
                args '-v /var/jenkins_home/maven/.m2:/root/.m2'               }
              steps {
                  echo 'Hello, Maven'
                   sh 'mvn --version'
               }
          }
          stage('Example Test') {
              agent { docker 'openjdk:8-jre' }
               steps {
                  echo 'Hello, JDK'
                  sh 'java -version'
               }
           }       }
 }
