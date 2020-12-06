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
   withEnv(['DOCKER_HOST=tcp://localhost:2375', 'PROJECT_NAME=jenkinsplayground', 'CONTAINER_REPO=algmprivsecops', 'DOCKER_SERVER=registry.hub.docker.com']) { 
     withCredentials([usernamePassword(credentialsId: 'CONTAINER_HUB_LOGIN', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
     
        container('alpine') {
          stage('Setup container env') {
            
            sh '''
              apk update \
              && apk --no-cache add git curl make python3 python3-dev gcc libc-dev libffi-dev py3-pip \
              openssl-dev \
              && pip3 --no-cache-dir install --upgrade pip \
              && pip3 --no-cache-dir install docker-compose==1.24.1 \
              && rm -f /var/cache/apk/* \
              && rm -rf /root/.cache
            '''
          }
         stage('Checkout Code') {         
            def checkout = checkout scm
            env.GIT_COMMIT = checkout["GIT_COMMIT"]
        } 
          stage('Build') {
            echo 'Building....'
            sh 'docker-compose --project-name $PROJECT_NAME build'
          }
           stage('Test in PR') {
        
           }
          stage('Deploy') {
            echo 'Deploying....'
            // def envvardeets = setLagoonEnvironmentVariables()
            // envvardeets.eachWithIndex{entry, i -> env[entry.key] = entry.value}
                dockerLogin()
                pushImageToRepo('node')
          }
        }
    }
   }
  }
}


// This function will try and achieve some level of parity with a lagoon
// build environment by translating Jenkins variables into the lagoon equivalents
def setLagoonEnvironmentVariables() {

    // def lagoonProject = getLagoonProjectName()

    // def lagoonBuildType = "branch"
    // if(thisIsAPullRequest()) {
    //     lagoonBuildType = "pullrequest"
    // }

    // def lagoonGitBranch = env.BRANCH_NAME
    // if(thisIsAPullRequest()) {
    //     lagoonGitBranch = env.CHANGE_BRANCH
    // }

    // def prTitle = ""
    // if(thisIsAPullRequest()) {
    //     prTitle = env.CHANGE_TITLE
    // }

    // def lagoonBaseBranch = ""
    // if(thisIsAPullRequest()) {
    //     lagoonBaseBranch = env.CHANGE_TARGET
    // }


    // return [LAGOON_PROJECT:lagoonProject,
    //         LAGOON_GIT_BRANCH:lagoonGitBranch ,
    //         LAGOON_BUILD_TYPE:lagoonBuildType,
    //         LAGOON_PR_BASE_BRANCH:lagoonBaseBranch,
    //         LAGOON_PR_TITLE:prTitle]
}


def dockerLogin() {
  sh "docker login -u$DOCKER_USERNAME -p$DOCKER_PASSWORD $DOCKER_SERVER"
}

def pushImageToRepo(imagename, tag = "latest") {

  sh '''
  images=$(docker images | grep "${PROJECT_NAME}"  | grep "${tag}" | cut -d" " -f1 | cat)
  for image in $(images); do
			docker tag ${PROJECT_NAME}_${image}:${tag} $DOCKER_SERVER/${CONTAINER_REPO}/${PROJECT_NAME}_${image}:${tag}
      docker push $DOCKER_SERVER/${CONTAINER_REPO}/${PROJECT_NAME}_${image}:${tag}
		done; 
	done
  '''
  
  //sh "docker tag ${PROJECT_NAME}_${imagename}:${tag} $DOCKER_SERVER/${CONTAINER_REPO}/${PROJECT_NAME}_${imagename}:${tag}"
  //sh "docker push $DOCKER_SERVER/${CONTAINER_REPO}/${PROJECT_NAME}_${imagename}:${tag}"
}

def getLagoonProjectName() {
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