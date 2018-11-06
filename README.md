# Installation

This is a simplified step-by-step installation of Red Hat OpenShift Service Mesh tech preview 3 on oc cluster.<br>
For additional instructions follow:<br>
https://docs.openshift.com/container-platform/3.11/servicemesh-install/servicemesh-install.html

## oc cluster download

Download:<br>
https://github.com/openshift/origin/releases/download/v3.11.0/openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz

## oc cluster patching

create a file:<br> *$HOME/.occluster/kube-apiserver/master-config.patch*<br>
with this content:<br>
```
admissionConfig:
  pluginConfig:
    MutatingAdmissionWebhook:
      configuration:
        apiVersion: apiserver.config.k8s.io/v1alpha1
        kubeConfigFile: /dev/null
        kind: WebhookAdmission
    ValidatingAdmissionWebhook:
      configuration:
        apiVersion: apiserver.config.k8s.io/v1alpha1
        kubeConfigFile: /dev/null
        kind: WebhookAdmission
```

execute:
```bash
$ cp -p master-config.yaml master-config.yaml.prepatch
$ oc ex config patch master-config.yaml.prepatch -p "$(cat master-config.patch)" > master-config.yaml
```

create a file:<br>
*/etc/sysctl.d/99-elasticsearch.conf*<br>
with this content:<br>
```
vm.max_map_count = 262144
```

execute:
```bash
$ sysctl vm.max_map_count=262144
```

## oc cluster up

```bash
$ oc cluster up --base-dir="$HOME/.occluster"
```

## istio operator

Download ocp resources from:<br>
https://github.com/Maistra/openshift-ansible/tree/maistra-0.3/istio

execute:<br>

```bash
$ oc new-project istio-operator
$ oc login -u system:admin
$ oc new-app -f istio_community_operator_template.yaml
```

create a istio-installation resource named *istio-installation.yaml*<br>with this content:<br>

```
apiVersion: "istio.openshift.com/v1alpha1"
kind: "Installation"
metadata:
  name: "istio-installation"
spec:
  deployment_type: openshift
  istio:
    authentication: true
    community: false
    prefix: openshift-istio-tech-preview/
    version: 0.3.0
  jaeger:
    prefix: distributed-tracing-tech-preview/
    version: 1.7.0
    elasticsearch_memory: 1Gi
  kiali:
    username: username
    password: password
    prefix: openshift-istio-tech-preview/
    version: 0.8.1
  launcher:
    openshift:
      user: user
      password: password
    github:
      username: username
      token: token
    catalog:
      filter: booster.mission.metadata.istio
      branch: v62
      repo: https://github.com/fabric8-launcher/launcher-booster-catalog.git
```

execute:

```bash
$ oc create -f istio-installation.yaml -n istio-operator
```

## istio control pane

execute:

```bash
$ oc create -f cr.yaml -n istio-operator
```

## verify the installation

execute:

```bash
$ oc get pods -n istio-system
```
