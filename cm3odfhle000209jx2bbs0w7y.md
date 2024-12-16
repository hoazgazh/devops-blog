---
title: "Running GitLab Runners on Kubernetes"
seoTitle: "Deploy GitLab Runners in Kubernetes"
seoDescription: "Learn how to set up GitLab Runners on Kubernetes and OpenShift using official Helm charts for optimized CI/CD pipelines"
datePublished: Tue Nov 19 2024 11:28:22 GMT+0000 (Coordinated Universal Time)
cuid: cm3odfhle000209jx2bbs0w7y
slug: running-gitlab-runners-on-kubernetes
tags: devops, helm, hoazgazh

---

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732807624462/d073dd0e-6534-439b-8a32-8ca63e4158d9.png align="center")

In this short article, we will explore how we can run Gitlab runners on Kubernetes using GitLab official helm charts with a bit of customization. GitLab Runners are integral components of GitLab’s Continuous Integration/Continuous Deployment (CI/CD) infrastructure, responsible for executing the defined tasks and workflows outlined in `.gitlab-ci.yml`configuration files. Now let's look at how to add one to your Kubernetes cluster.

## **Pre-requisites**

* Kubernetes Cluster
    
* Helm CLI
    

## **Setup with** kubernetes or EKS

1. The first step is to add the helm charts via helm cli
    

```plaintext
# Add Chart
helm repo add gitlab https://charts.gitlab.io
helm repo update
```

2\. To register a Gitlab Runner with your GitLab Instance, it needs a registration token, which can be created from the below URL. Please note that I am using GitLab.com instance and all my projects are under a particular group. Don't forget to copy the registration token after the creation :)

[https://gitlab.com/groups/&lt;groupname&gt;/-/runners](https://gitlab.com/groups/edgeworks2/-/runners)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732807635914/147dae4c-e72f-4281-9756-4dce0cfde946.png align="center")

GitLab Runner Registration

3\. Install the chart specifying the registration token, version, and values file

```plaintext
helm install --namespace gitlab-runner --create-namespace --set runnerRegistrationToken=<replacewithyourtoken>  gitlab-runner gitlab/gitlab-runner --version v0.63.0  --values values.yaml
```

*values.yaml*

```plaintext
gitlabUrl: https://gitlab.com/

imagePullPolicy: IfNotPresent
concurrent:  4

imagePullSecrets:
  - name: harbor-pull-secret

replicas: 5

rbac:
  create: true
  serviceAccountName: default

runners:
  config: |
    [[runners]]
      name = "gitlab-runner"
      executor="kubernetes"
      environment = [
        "FF_KUBERNETES_HONOR_ENTRYPOINT=false",
        "FF_USE_LEGACY_KUBERNETES_EXECUTION_STRATEGY=true",
        ]
      [runners.kubernetes]
         poll_timeout = 2000
         node_selector_overwrite_allowed = ".*"
         helper_image = "gitlab/gitlab-runner-helper:arm64-v16.10.0"
         image_pull_secrets=["harbor-pull-secret"]

unregisterRunners: true

securityContext:
  allowPrivilegeEscalation: true
  readOnlyRootFilesystem: false
  runAsNonRoot: true
  privileged: true
  capabilities:
    drop: ["ALL"]
```

4\. The above value file has been used to deploy the chart on the arm64 Kubernetes nodes cluster and that is the reason the helper image used is with `arm64` tag. The imagePullSecrets contains the name of the secret that the `.dockerconfigjson,` will be used to pull images from the external container registry. You may create one, by reading the steps [here](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

That’s all for now. Thanks for reading and feedback is always welcome. Until next time.

In case of any queries, please feel free to connect with me via the below social links

# Setup with Openshift

### Description

This repository contains Helm chart for deploying GitLab runner on OpenShift.

This Helm chart creates a GitLab Multi-Runner manager in your OpenShift project.

It defaults to executing GitLab CI jobs using the multi-runner's kubernetes executor.

By deploying the multi-runner in OpenShift, it is able to automatically detect and use the OpenShift kubernetes config to create new pods for CI job execution.

#### Service Account User

The `service-account.yaml` file creates a service user for the runner application. The service user's name is `gitlab-runner-user`.

This user requires the ability to run as privileged container in order to support dind builds. In order to run the Runner application. In OpenShift this means an administrator needs to add the user to the `privileged` security context.

They can do this by running:

```plaintext
$ oc adm policy add-scc-to-user privileged system:serviceaccount:<project-name>:gitlab-runner-user
```

The Runner application deployment will fail to successfully start until this has been done.

### Setup

1. Add the policy as described in previous section.
    
2. Log in to OpenShift cluster
    

```plaintext
$ oc login <server-url> --token=<token>
```

3. Provide `gitlabUrl` and `runnerRegistrationToken` in `values.yaml`.
    
4. Install helm chart
    

```plaintext
$ helm install gitlab-runner .\gitlab-runner-openshift --namespace <project-name>
```

### Chart values

```plaintext

imageName: gitlab/gitlab-runner
imageTag: alpine-v11.5.1

imagePullPolicy: IfNotPresent

gitlabUrl: ""

runnerRegistrationToken: ""

concurrent: 10

checkInterval: 30

runners:
  image: ubuntu:16.04
  privileged: true
  config: |
    [[runners]]
      builds_dir = "/tmp"
      environment = ["HOME=/tmp"]
      [runners.kubernetes]
        privileged = false

securityContext:
  runAsNonRoot: true

rbac:
  create: true

serviceAccount:
  create: true
  name: default
```