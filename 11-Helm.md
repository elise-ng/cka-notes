# Helm

## Basic YAML
- We can define all related resources (e.g. Service, Deployment, PV, PVC) of an application in one YAML file
- Use separators (`---`) between components
- This works but not portable -> we can refactor parameters from the generic application config using Helm

