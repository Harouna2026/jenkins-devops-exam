pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "harounacsk" // replace this with your docker-id
DOCKER_IMAGE1= "movie-service"
DOCKER_IMAGE2= "cast-service"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
  stage(' Docker Build'){ // docker build image stage
    steps {
      script {
      sh '''
        docker rm -f jenkins
        export VERSION="${DOCKER_TAG}"
        docker compose build 
      sleep 6
      '''
      }
    }
  }
 
 
  stage(' Docker run'){ // docker build image stage
    steps {
      script {
      sh '''
        docker rm -f jenkins
        docker compose up -d 
      sleep 6
      '''
      }
    }
  }

  stage('Docker Push'){ //we pass the built image to our docker hub account
    environment
    {
      DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
    }
    steps {
      script {
        sh '''
        docker login -u $DOCKER_ID -p $DOCKER_PASS
        export VERSION="${DOCKER_TAG}"
        docker compose push
        '''
      }
    }
  }
  stage('Deploiement en dev'){
    environment {
    KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
    }
    steps {
      script {
      sh '''
      rm -Rf ~/.kube/
      mkdir ~/.kube/
      ls
      cat $KUBECONFIG > ~/.kube/config
      cp charts/values.yaml values.yml
      cat values.yml
      sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
        helm upgrade --install app-movie charts --values=values.yml --namespace dev \
        --set image.repository="${DOCKER_ID}/${DOCKER_IMAGE1}" \
        --set image.tag="${DOCKER_TAG}" \
        --set service.nodePort=30001

         helm upgrade --install app-cast charts --values=values.yml --namespace dev \
        --set image.repository="${DOCKER_ID}/${DOCKER_IMAGE1}" \
        --set image.tag="${DOCKER_TAG}" \
        --set service.nodePort=30002
      '''
      }
    }
  }
  stage('Deploiement en staging'){
    environment {
    KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
    }
    steps {
      script {
      sh '''
      rm -Rf ~/.kube/
      mkdir ~/.kube/
      ls
      cat $KUBECONFIG > ~/.kube/config
      cp charts/values.yaml values.yml
      cat values.yml
      sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
      helm upgrade --install app-movie movie-service --values=values.yml --namespace staging \
        --set image.repository="${DOCKER_ID}/${DOCKER_IMAGE1}" \
        --set image.tag="${DOCKER_TAG }" \
        --set service.nodePort=30001

         helm upgrade --install app-cast charts --values=values.yml --namespace staging \
        --set image.repository="${DOCKER_ID}/${DOCKER_IMAGE1}" \
        --set image.tag="${DOCKER_TAG}" \
        --set service.nodePort=30002
      '''
      }
    }
  }
    stage('Deploiement en qa'){
    environment {
    KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
    }
    steps {
      script {
      sh '''
      rm -Rf ~/.kube/
      mkdir ~/.kube/
      ls
      cat $KUBECONFIG > ~/.kube/config
      cp charts/values.yaml values.yml
      cat values.yml
      sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
      helm upgrade --install app-movie movie-service --values=values.yml --namespace qa \
        --set image.repository="${DOCKER_ID}/${DOCKER_IMAGE1}" \
        --set image.tag="${DOCKER_TAG }" \
        --set service.nodePort=30001

         helm upgrade --install app-cast charts --values=values.yml --namespace qa \
        --set image.repository="${DOCKER_ID}/${DOCKER_IMAGE1}" \
        --set image.tag="${DOCKER_TAG}" \
        --set service.nodePort=30002
      '''
      }
    }
  }
  stage('Deploiement en prod'){
    environment {
      KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
      }
      steps {
        // Create an Approval Button with a timeout of 15minutes.
        // this require a manuel validation in order to deploy on production environment
        timeout(time: 15, unit: "MINUTES") {
          input message: 'Do you want to deploy in production ?', ok: 'Yes'
        }
        script {
          sh '''
          rm -Rf ~/.kube/
          mkdir ~/.kube/
          ls
          cat $KUBECONFIG > ~/.kube/config
          cp charts/values.yaml values.yml
          cat values.yml
          sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
          helm upgrade --install app-movie movie-service --values=values.yml --namespace prod \
            --set image.repository="${DOCKER_ID}/${DOCKER_IMAGE1}" \
            --set image.tag="${DOCKER_TAG }" \
            --set service.nodePort=30001
          helm upgrade --install app-cast charts --values=values.yml --namespace prod \
            --set image.repository="${DOCKER_ID}/${DOCKER_IMAGE1}" \
            --set image.tag="${DOCKER_TAG}" \
            --set service.nodePort=30002
          '''
        }
      }
    }
  }
}