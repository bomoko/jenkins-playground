#!/usr/bin/env groovy

podTemplate(
  nodeSelector:"lagoon/build=allowed",
  yaml: """
spec:
  tolerations:
    - effect: NoSchedule
      key: lagoon/build
      operator: Exists
    - effect: PreferNoSchedule
      key: lagoon/build
      operator: Exists
    - effect: NoSchedule
      key: lagoon.sh/build
      operator: Exists
    - effect: PreferNoSchedule
      key: lagoon.sh/build
      operator: Exists
""",
  annotations: [
    podAnnotation(key: "cluster-autoscaler.kubernetes.io/safe-to-evict", value: "false")
  ],
  containers: [
    containerTemplate(
      name: 'dindcontainer',
      image: 'docker:dind',
      ttyEnabled: true,
      privileged: true,
      envVars: [envVar(key: 'DOCKER_TLS_CERTDIR', value: '')],
      resourceRequestCpu: '4',
      resourceRequestMemory: '16Gi'
      ),
    containerTemplate(
      name: 'alpine',
      image: 'docker:latest',
      ttyEnabled: true,
      command: 'cat',
      resourceRequestCpu: '100m',
      resourceRequestMemory: '200Mi'
      ),
    ],
    ) {
  node(POD_LABEL) {
   withEnv(['DOCKER_HOST=tcp://localhost:2375']) {
node {
    stage('Build') {
        echo 'Building....'
    }
    stage('Test in PR') {
        sh "env | sort"
    }
    stage('Deploy') {
        echo 'Deploying....'
        def envvardeets = setLagoonEnvironmentVariables()
        echo envvardeets.join(":")
    }
}

   }
  }
}


// This function will try and achieve some level of parity with a lagoon
// build environment by translating Jenkins variables into the lagoon equivalents
def setLagoonEnvironmentVariables() {

    define lagoonProject = getLagoonProjectName()

    def lagoonBuildType = "branch"
    if(thisIsAPullRequest()) {
        lagoonBuildType = "pullrequest"
    }

    def lagoonGitBranch = env.BRANCH_NAME
    if(thisIsAPullRequest()) {
        lagoonGitBranch = env.CHANGE_BRANCH
    }

    def prTitle = ""
    if(thisIsAPullRequest()) {
        prTitle = env.CHANGE_TITLE
    }

    def lagoonBaseBranch = ""
    if(thisIsAPullRequest()) {
        lagoonBaseBranch = env.CHANGE_TARGET
    }


    return ['LAGOON_PROJECT='+lagoonProject,
            'LAGOON_GIT_BRANCH='+lagoonGitBranch ,
            'LAGOON_BUILD_TYPE=' + lagoonBuildType,
            'LAGOON_PR_BASE_BRANCH=' + lagoonBaseBranch,
            'LAGOON_PR_TITLE=' + prTitle]
}


define getLagoonProjectName() {
    return "lagoonProjectName"
}

def getProjectNameFromPRTitle(prTitle) {
  if(prTitle ==~ /^BEO_.*$/) {
    return "test"
  }

  return false
}



def thisIsAPullRequest() {
  if (env.CHANGE_ID) {
   return true
  }
return false
}