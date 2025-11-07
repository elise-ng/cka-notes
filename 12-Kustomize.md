# Kustomize

https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/

- k8s feature to use file `kustomization.yaml` to apply changes to a set of resources
- convenient for apply changes to input files that user does not control
- `kubectl apply -k ./` in directory with the file
- `kubectl delete -k ./`

## Sample

Below same applies overrides on the listed resources yaml

```yaml
resources:
- deployment.yaml
- service.yaml
namePrefix: test-
namespace: testing
commonLabels:
  environment: testing
```

Below defines a base config, plus multiple deployment overlays e.g. dev, staging, prod

`kustomization.yaml`
```yaml
- base
  - deployment.yaml
  - service.yaml
  - kustomization.yaml
- overlays
  - dev
    - kustomization.yaml
  - staging
    - kustomization.yaml
  - prod
    - kustomization.yaml
```

`overlays/{dev,staging,prod}/kustomization.yaml`
```yaml
resources:
- ../../base
namePrefix: dev-
namespace: development
commonLabels:
  environment: development
```
