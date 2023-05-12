pipeline {
  agent any
  stages {
    stage('Docker Build') {
      steps {
        script {
          sh '''
sudo docker rm -f jenkins
sudo docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
sleep 6
'''
        }

      }
    }

    stage('Docker run') {
      steps {
        script {
          sh '''
docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
sleep 10
'''
        }

      }
    }

    stage('Test Acceptance') {
      steps {
        script {
          sh '''
curl localhost
'''
        }

      }
    }

    stage('Docker Push') {
      environment {
        DOCKER_PASS = credentials('DOCKER_HUB_PASS')
      }
      steps {
        script {
          sh '''
docker login -u $DOCKER_ID -p $DOCKER_PASS
docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
'''
        }

      }
    }

    stage('Deploiement en dev') {
      environment {
        KUBECONFIG = credentials('config')
      }
      steps {
        script {
          sh '''
rm -Rf .kube
mkdir .kube
ls
cat $KUBECONFIG > .kube/config
cp fastapi/values.yaml values.yml
cat values.yml
sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
helm upgrade --install app fastapi --values=values.yml --namespace dev
'''
        }

      }
    }

    stage('Deploiement en staging') {
      environment {
        KUBECONFIG = credentials('config')
      }
      steps {
        script {
          sh '''
rm -Rf .kube
mkdir .kube
ls
cat $KUBECONFIG > .kube/config
cp fastapi/values.yaml values.yml
cat values.yml
sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
helm upgrade --install app fastapi --values=values.yml --namespace staging
'''
        }

      }
    }

    stage('Deploiement en prod') {
      environment {
        KUBECONFIG = credentials('config')
      }
      steps {
        timeout(time: 15, unit: 'MINUTES') {
          input(message: 'Do you want to deploy in production ?', ok: 'Yes')
        }

        script {
          sh '''
rm -Rf .kube
mkdir .kube
ls
cat $KUBECONFIG > .kube/config
cp fastapi/values.yaml values.yml
cat values.yml
sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
helm upgrade --install app fastapi --values=values.yml --namespace prod
'''
        }

      }
    }

  }
  environment {
    DOCKER_ID = 'abrarhm'
    DOCKER_IMAGE = 'datascientestapi'
    DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
  }
}