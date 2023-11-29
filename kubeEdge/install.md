### Specs
- Ubuntu20.04 VM
- k8s version -> v1.25.5
- KubeEdge version -> 1.15

## containerd :
- https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd
- https://github.com/containerd/containerd/blob/main/docs/getting-started.md

```
sudo apt-get update
sudo apt-get install containerd.io
sudo mv /etc/containerd/config.toml /etc/containerd/config.toml.old
containerd config default | sudo tee /etc/containerd/config.toml
```





