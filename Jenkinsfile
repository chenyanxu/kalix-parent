#!/usr/bin/groovy
def localItestPattern = ""
try {
  localItestPattern = ITEST_PATTERN
} catch (Throwable e) {
  localItestPattern = "*KT"
}
def localFailIfNoTests = ""
try {
  localFailIfNoTests = ITEST_FAIL_IF_NO_TEST
} catch (Throwable e) {
  localFailIfNoTests = "false"
}
def versionPrefix = ""
try {
  versionPrefix = VERSION_PREFIX
} catch (Throwable e) {
  versionPrefix = "1.0"
}
def canaryVersion = "${versionPrefix}.${env.BUILD_NUMBER}"
def fabric8Console = "${env.FABRIC8_CONSOLE ?: ''}"
def utils = new io.fabric8.Utils()
node {
  def envStage = utils.environmentNamespace('staging')
  def envProd = utils.environmentNamespace('production')
  git 'http://gogs.apps.kalix.com/gogsadmin/trycamel.git'
  echo 'NOTE: running pipelines for the first time will take longer as build and base docker images are pulled onto the node'
  kubernetes.pod('buildpod').withImage('fabric8/maven-builder')
      .withPrivileged(true)
      .withHostPathMount('/var/run/docker.sock','/var/run/docker.sock')
      .withHostPathMount('/root/.mvnrepository','/var/lib/maven/repository')
      .withEnvVar('DOCKER_CONFIG','/home/jenkins/.docker/')
      .withSecret('jenkins-docker-cfg','/home/jenkins/.docker')
      .withSecret('jenkins-maven-settings','/root/.m2')
      .withServiceAccount('jenkins')
      .inside {
    stage 'Canary Release'
    mavenCanaryRelease{
      version = canaryVersion
    }
    stage 'Integration Testing'
    mavenIntegrationTest{
      environment = 'Testing'
      failIfNoTests = localFailIfNoTests
      itestPattern = localItestPattern
    }
    stage 'Rolling Upgrade Staging'
    kubernetesApply(environment: envStage)
    stage 'Approve'
    approve{
      room = null
      version = canaryVersion
      console = fabric8Console
      environment = 'Staging'
    }
    stage 'Rolling Upgrade Production'
    kubernetesApply(environment: envProd)
  }
}