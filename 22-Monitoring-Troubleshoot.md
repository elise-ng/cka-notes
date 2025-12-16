# Logging & Monitoring

- `kubectl get`: generic resource health
- `kubectl top pods`, `kubectl top nodes`: performance info if metrics-server installed
- advanced tools e.g. Prometheus, Grafana

# Troubleshooting 

- resources first created in etcd database
- `kubectl describe`, `kubectl events`: check resource creation status
- after resource added to db, pod is started on scheduled nodes
- before pod can be started, container image need to be fetched
  - `sudo crictl images`
- `kubectl logs` once pod started
- `kubectl exec -it <pod> -- sh`: start interactive shell
  - if `ps` not available, `cat /proc/1/cmdline` and `cat /proc/1/status`

## Flow
- `kuebctl create` -> etcd -> scheduler: error message available at `kubectl describe`, `kubectl events`
- scheduler -> kubelet -> cri (pull image) -> (container started): `crictl inspect`, `kubectl logs`

## Troubleshoot Applications
- `kubectl get`: show pod states
  - Pending: created in etcd but waiting for eligible node
  - Running: healthy
  - Succeeded: done work and no need to restart
  - Failed: pod ended in error code, will not be restarted
  - Unknown: state cannot be obtained, usually network issues
  - Completed: run to completion
  - CrashLookBackOff: ended in error but still trying to restart
- `kubectl describe`: shows API info about resource
- `kubectl log`
- `kubectl exec -it mypod -- sh`

## Troubleshoot Cluster Nodes
- `kubectl cluster-info`: show cluster health overview
- `kubectl cluster-info dump`: verbose info from all cluster log files
- `kubectl get nodes`: shows node health overview
- `kubectl get pods -n kube-system`: check core services
  - `kube-proxy` pods for connectivity with worker nodes
- `kubectl describe node mynode`: show details of node, check `Conditions` section
- `sudo systemctl status kubelet`: show kubelet status info
  - `sudo systemctl restart kubelet`
- `sudo openssl x509 -in /var/lib/kubelet/pki/kubelet.crt -text`: verify kubelet certs still valid
