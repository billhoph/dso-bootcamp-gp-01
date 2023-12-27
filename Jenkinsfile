pipeline {
  agent any
  stages {
    stage('Git Checkout') {
      parallel {
        stage('Git Checkout') {
          steps {
            sh '''rm -Rf ./dso-bootcamp-gp-01

git clone https://github.com/billhoph/dso-bootcamp-gp-01.git
ls ./dso-bootcamp-gp-01'''
          }
        }

        stage('Clean up Images on Node') {
          steps {
            sh 'if docker image inspect harbor.alson.space/jenkins/yelb-ui:1.1 >/dev/null 2>&1; then docker rmi harbor.alson.space/jenkins/yelb-ui:1.1; fi'
            sh 'if docker image inspect harbor.alson.space/jenkins/yelb-db:1.1 >/dev/null 2>&1; then docker rmi harbor.alson.space/jenkins/yelb-db:1.1; fi'
            sh 'if docker image inspect harbor.alson.space/jenkins/yelb-appserver:1.1 >/dev/null 2>&1; then docker rmi harbor.alson.space/jenkins/yelb-appserver:1.1; fi'
          }
        }

      }
    }

    stage('Code Scanning') {
      steps {
        sh 'checkov -d ./dso-bootcamp-gp-01 --use-enforcement-rules --prisma-api-url https://api.sg.prismacloud.io --bc-api-key $pcskey --repo-id dso-bootcamp-gp-01/code-checking --branch main -o cli --quiet --compact'
      }
    }

    stage('Docker Image Building') {
      parallel {
        stage('Building UI app Image') {
          steps {
            sh 'docker build ./dso-bootcamp-gp-01/yelb-ui -t harbor.alson.space/jenkins/yelb-ui:1.1'
          }
        }

        stage('Building DB Image') {
          steps {
            sh 'docker build ./dso-bootcamp-gp-01/yelb-db -t harbor.alson.space/jenkins/yelb-db:1.1'
          }
        }

        stage('Building AppServer Image') {
          steps {
            sh 'docker build ./dso-bootcamp-gp-01/yelb-appserver -t harbor.alson.space/jenkins/yelb-appserver:1.1'
          }
        }

      }
    }

    stage('List Successful Build Images') {
      steps {
        sh 'docker images'
      }
    }

    stage('Scanning Images (UI Image)') {
      parallel {
        stage('Scanning Images (UI Image)') {
          steps {
            prismaCloudScanImage(image: 'harbor.alson.space/jenkins/yelb-ui:1.1', resultsFile: './yelb-ui-scan.json', logLevel: 'info', dockerAddress: 'unix:///var/run/docker.sock')
          }
        }

        stage('Scanning Images (App Image)') {
          steps {
            prismaCloudScanImage(image: 'harbor.alson.space/jenkins/yelb-appserver:1.1', resultsFile: './yelb-app-scan.json', logLevel: 'info', dockerAddress: 'unix:///var/run/docker.sock')
          }
        }

        stage('Scanning Images (DB Image)') {
          steps {
            prismaCloudScanImage(dockerAddress: 'unix:///var/run/docker.sock', image: 'harbor.alson.space/jenkins/yelb-db:1.1', resultsFile: './yelb-db-scan.json', logLevel: 'info')
          }
        }

      }
    }

    stage('Publish Scan Result') {
      steps {
        sh 'ls -tlr'
        prismaCloudPublish(resultsFilePattern: '*.json')
      }
    }

    stage('Push Images to Harbor') {
      steps {
        sh '''docker login harbor.alson.space -u admin -p Harbor12345
docker push harbor.alson.space/jenkins/yelb-ui:1.1
docker push harbor.alson.space/jenkins/yelb-db:1.1
docker push harbor.alson.space/jenkins/yelb-appserver:1.1
'''
      }
    }

    stage('Deploy Application') {
      steps {
        sh '''kubectl cluster-info --context kind-demo
kubectl get pod -A
kubectl delete ns dso-bootcamp-gp-01
kubectl create ns dso-bootcamp-gp-01
kubectl apply -f ./dso-bootcamp-gp-01/deployments/platformdeployment/Kubernetes/yaml/yelb-k8s-minikube-nodeport.yaml -n dso-bootcamp-gp-01
kubectl get svc/yelb-ui -n dso-bootcamp-gp-01'''
      }
    }

  }
  environment {
    pcskey = '1c788e25-83ba-4b31-94d6-3f99234e2e46::DdgP+op52pyjTlAF+nnpkXxt0i4='
  }
}