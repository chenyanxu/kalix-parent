#!/usr/bin/groovy
node{
  git 'https://github.com/minikiller/kalix-parent.git'
  withEnv(["PATH+MAVEN=${tool 'maven-3.3.1'}/bin"]) {

    stage 'Deploy'
    sh 'mvn clean install org.apache.maven.plugins:maven-deploy-plugin:2.8.2:deploy'
  }
}
