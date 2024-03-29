stage('Configure') {
    abort = false
    inputConfig = input id: 'InputConfig', message: 'Docker registry and Anchore Engine configuration', parameters: [string(defaultValue: 'https://index.docker.io/v1/', description: 'URL of the docker registry for staging images before analysis', name: 'dockerRegistryUrl', trim: true), string(defaultValue: 'docker.io', description: 'Hostname of the docker registry', name: 'dockerRegistryHostname', trim: true), string(defaultValue: '', description: 'Name of the docker repository', name: 'dockerRepository', trim: true), credentials(credentialType: 'com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl', defaultValue: '', description: 'Credentials for connecting to the docker registry', name: 'dockerCredentials', required: true), string(defaultValue: '', description: 'Anchore Engine API endpoint', name: 'anchoreEngineUrl', trim: true), credentials(credentialType: 'com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl', defaultValue: '', description: 'Credentials for interacting with Anchore Engine', name: 'anchoreEngineCredentials', required: true)]

    for (config in inputConfig) {
        if (null == config.value || config.value.length() <= 0) {
          echo "${config.key} cannot be left blank"
          abort = true
        }
    }
}

node {
  def app
  def dockerfile
  def anchorefile

  try {
      stage('Checkout') {
        // Clone the git repository
        checkout scm
        def path = sh returnStdout: true, script: "pwd"
        path = path.trim()
        dockerfile = path + "/Dockerfile"
        anchorefile = path + "/anchore_images"
      }

      stage('Parallel') {
        parallel Test: {
          app.inside {
              sh 'echo "Dummy - tests passed"'
          }
        },
        Analyze: {
        sh 'echo "docker.io/dhaval3905/centos-openjdk-11:latest `pwd`/Dockerfile" > anchore_images'
              anchore bailOnFail: false, bailOnPluginFail: false, name: 'anchore_images'
        }
      }
  } finally {
    stage('Cleanup') {
      // Delete the docker image and clean up any allotted resources
      sh script: "docker rmi " + repotag
    }
  }
}
