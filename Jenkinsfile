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
    }
}

   }
  }
}
