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
