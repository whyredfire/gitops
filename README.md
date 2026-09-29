# gitops

Argo CD manifests for a single-node k3s cluster.

## Node configuration

These live on the node, not in this repo, so reapply them when rebuilding it.

### k3s

The bundled Traefik is replaced by the Argo CD-managed one, which is exposed
via `externalIPs` rather than ServiceLB. Control-plane metrics endpoints are
exposed for Prometheus:

```yaml
# /etc/rancher/k3s/config.yaml
disable:
  - traefik
  - servicelb
kube-controller-manager-arg:
  - bind-address=0.0.0.0
kube-scheduler-arg:
  - bind-address=0.0.0.0
kube-proxy-arg:
  - metrics-bind-address=0.0.0.0
```

### Image garbage collection

Kubelet only prunes images once the disk crosses 85%, so unused images pile up
on a large disk. Also expire images that haven't been used in 7 days:

```yaml
# /var/lib/rancher/k3s/agent/etc/kubelet.conf.d/50-image-gc.conf
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
imageMaximumGCAge: 168h
```

Apply with `sudo systemctl restart k3s`.
