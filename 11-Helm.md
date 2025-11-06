# Helm

## Basic YAML
- We can define all related resources (e.g. Service, Deployment, PV, PVC) of an application in one YAML file
- Use separators (`---`) between components
- This works but not portable -> we can refactor parameters from the generic application config using Helm

## Helm info
- Included in CKA exam even though Helm is an ecosystem package
- We need the helm cli tool to use it
  - https://github.com/helm/helm/releases
- Chart
  - a Helm package
  - contains description and templates of k8s manifest files
  - can be stored locally or access from remote repos
- Main site for helm charts: https://artifacthub.io/

### Usual usage

```sh
helm repo add <repo name> <repo url>
helm install <app name> <chart name>
```

### Helm Demo
```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
# show added repos
helm repo list
# search packages on repo
helm search repo nginx
# search for versions
helm search repo nginx --versions

# sync local with repo
helm repo update
# test install mysql
helm install bitnami/mysql --generate-name
# list installed charts
helm list
# check install status
helm status mysql-1762393608
```

## Creating a template from a helm chart
- We can render a yaml file without deploying the helm chart to cluster
- Then we can install by `kubectl apply`

### Helm Template Demo

```sh
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm search repo argo/argo-cd
helm template my-argo-cd argo/argo-cd --version 9.1.0 > argo-cd-template.yaml
# the yaml template is now at argo-cd-template.yaml

# to render with values
# get default values.yaml
helm show values argo/argo-cd > values.yaml
# edit values, then
helm template my-argo-cd argo/argo-cd -f values.yaml > argo-cd-template.yaml
```

## Helm operations
- We will put custom values at `values.yaml`, which will be merged with the defaults in helm chart
- Check documentation for configurable values
- `helm install <...> --values values.yaml` (or `--set key=value`, not recommended)
- `helm upgrade <...> --values values.yaml`: we need to supply the values.yaml every time
- `helm get values <app name>`: show user supplied values of an release
- A Helm release is combination of chart and installation
  - we can `helm upgrade` to change deployed chart version and/or template values
