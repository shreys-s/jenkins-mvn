node {
  def branchVersion = ""

  try {
    
withDockerContainer('jenkins-slave-image3') {

    stage ('Checkout') {
      // checkout repository
      checkout scm

    }

    stage ('Determine Branch Version') {
      // determine version in pom.xml
      def pomVersion = sh(script: 'mvn -q -Dexec.executable=\'echo\' -Dexec.args=\'${project.version}\' --non-recursive exec:exec', returnStdout: true).trim()

      // compute proper branch SNAPSHOT version
      pomVersion = pomVersion.replaceAll(/-SNAPSHOT/, "") 
      branchVersion = env.BRANCH_NAME
      branchVersion = branchVersion.replaceAll(/origin\//, "") 
      branchVersion = branchVersion.replaceAll(/\W/, "-")
      branchVersion = "${pomVersion}-${branchVersion}-SNAPSHOT"

      // set branch SNAPSHOT version in pom.xml
      sh "mvn versions:set -DnewVersion=${branchVersion}"
    	}

    stage ('Java Build') {
      // build .war package
      sh 'mvn clean package -U'
    }

}
  
    stage ('Docker Build') {
      // prepare docker build context
      sh "cp target/project.war ./tmp-docker-build-context"

      // Build and push image with Jenkins' docker-plugin
      withDockerServer([uri: "tcp://<my-docker-socket>"]) {
        withDockerRegistry([credentialsId: 'docker-registry-credentials', url: "https://<my-docker-registry>/"]) {
          // we give the image the same version as the .war package
          def image = docker.build("<myDockerRegistry>/<myDockerProjectRepo>:${branchVersion}", "--build-arg PACKAGE_VERSION=${branchVersion} ./tmp-docker-build-context")
          image.push()
        }   
      }
    } 
  } catch(e) {
    currentBuild.result = "FAILED"
    throw e
  } finally {
  }
}
