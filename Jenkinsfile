node {
  properties([
    [$class: 'BuildDiscarderProperty', 
      strategy: [
        $class: 'LogRotator', 
        artifactNumToKeepStr: '5', 
        daysToKeepStr: '30']
    ],
    disableConcurrentBuilds(),
    rateLimitBuilds([count: 1, durationName: 'minute', userBoost: false]),
    pipelineTriggers([upstream(upstreamProjects: 'hop', threshold: hudson.model.Result.SUCCESS)]),
    parameters([
     string(name: 'PRM_BRANCHNAME', defaultValue: "master"),
     string(name: 'PRM_BUILD_NUMBER', defaultValue: "0"),
    ]),
  ])

  try{
    stage('Checkout') {
      checkout scm
    }

    stage('Upstream Variables') {
      echo "upstream Branch: ${params.PRM_BRANCHNAME}"
      echo "upstream Build Number: ${params.PRM_BUILD_NUMBER}"
    }


    stage('Build image') {
      docker.withRegistry('', 'dockerhub') {
        if("${params.PRM_BRANCHNAME}" == "master"){

          def customImage = docker.build("projecthop/hop:snapshot" , "--build-arg build_number=${params.PRM_BUILD_NUMBER} .")
          customImage.push()
        } else 
        {

          def customImage = docker.build("projecthop/hop:${params.PRM_BRANCHNAME}", "--build-arg build_number=${params.PRM_BUILD_NUMBER} .")
          customImage.push()
        }
    
        /* Push the container to the custom Registry */
        
      }
  }

    stage('Cleanup'){
      if("${params.PRM_BRANCHNAME}" == "master"){

        sh 'docker rmi projecthop/hop:snapshot'
      }
      else
      {
        sh "docker rmi projecthop/hop:${params.PRM_BRANCHNAME}"
      }
    }

  } finally 
  {
    cleanWs()
  }
}