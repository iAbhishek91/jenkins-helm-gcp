# Jenkins on GCP

This repo demonstrate hosting of jenkins on kubernetes cluster. Its need to be AWS or GKE

> NOTE: Local cluster most probably will not work as LoadBalancer service type is used and LoadBalancer is not configured in most of the local k8s cluster.

## Steps to deploy

Every deploy will create fresh jenkins instance with persistent volume.

For GCP, please execute the below commands in cloud shell. *Helm* and *kubectl* will be pre-installed in containers.

```sh
# fetch the kubeconfig in cloud shell
gcloud container clusters get-credentials <cluster-name> --zone <zone-name> --project <project-id>
```

helm commands below:

```sh
## validate the helm chart
helm template ./jenkins

## deploy the helm chart
helm install gcp ./jenkins

## further changes in the jenkins you may delete upgrade the help deployment
helm upgrade gcp ./jenkins
```

## Configure Jenkins

Once Started configure the cloud.

### Kubernetes cloud

Name: kubernetes // any name is fine
Kubernetes URL: https://<ip>:<port> // cluter IP from kubeconfig file
kubernetes Namespace: default // used to host the slave agent
Default provider template name: jnlp // this is the default container where the build will execute.
Pod template:
  Name: jnlp
  Namespace: default
  Usage: Only build jobs with label expression matching this node
  Container template:
    Name: jnlp
    Docker image: jenkins/jnlp-slave
    Working dir: /home/jenkins/agent

### Configure a test job to validate

Create a new item as pipeline and name it as test-pipeline-job

Pipeline script

```yaml
pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    some-label: some-label-value
spec:
  containers:
  - name: maven
    image: maven:alpine
    command:
    - cat
    tty: true
  - name: busybox
    image: busybox
    command:
    - cat
    tty: true
"""
    }
  }
  stages {
    stage('Run maven') {
      steps {
        container('maven') {
          sh 'mvn -version'
        }
        container('busybox') {
          sh '/bin/busybox'
        }
      }
    }
  }
}
```
