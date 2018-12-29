def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(label: label,
     containers: [
        containerTemplate(name: 'sigma-agent', image: 'invent360/sigma-agent:latest',ttyEnabled: true, command: 'cat'),
     ],
     volumes: [
        hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')
  ]) {
    node(label) {

        checkout scm

        dir('angular-k8') {
            git url: 'https://github.com/katson95/angular-k8.git'
        }

        def IMAGE_NAME = 'invent360/ng-app'
        def IMAGE_VERSION = 'latest'

       stage('Get a Angular project') {
            container('sigma-agent') {
                stage('Build a Angular project') {
                    sh 'npm install'
                    sh 'ls -ltr'
                    sh 'ng build'
                }
            }
        }

        stage('Build and Test Image') {
            container('sigma-agent') {
                stage('Package into Docker Image') {
                    sh 'docker build -t ng-app:latest .'
                    sh 'docker tag ng-app:latest docker.ops.dev.invent-360.com/invent360/ng-app:latest'
                }
            }
        }

        stage('Publish Image') {
            container('sigma-agent'){
                stage('Publish Image to Docker Registry') {
                  withCredentials([usernamePassword(credentialsId: 'i360-nexus-id', passwordVariable: 'dockerPassword', usernameVariable: 'dockerUsername')]) {
                   sh "docker login -u ${env.dockerUsername} -p ${env.dockerPassword} docker.ops.dev.invent-360.com"
                   sh "docker push docker.ops.dev.invent-360.com/${IMAGE_NAME}:${IMAGE_VERSION}"
                }
              }
            }
        }

        stage('Deploy To Dev') {
            container('sigma-agent'){
                stage('Deploy To Dev') {
                    sh 'kubectl create ns DEV'
                    sh 'kubectl create -f ./angular-k8/ --namespace=DEV'
              }
            }
        }
    }
}
