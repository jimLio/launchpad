### Testbed
- Ubuntu 20.04 VM
- k8s version -> v1.25.5
- KubeEdge version -> 1.12.6

### Prerequisites 
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

### containerd  
Resources:<br>
- [kubernetes docs](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)
- [containerd github](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)
- [kubeEdge docs](https://release-1-15.docs.kubeedge.io/docs/setup/prerequisites/runtime)

First add [Docker apt repo](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)  

```
sudo apt-get install containerd.io
sudo mv /etc/containerd/config.toml /etc/containerd/config.toml.old

# enable cri plugin
containerd config default | sudo tee /etc/containerd/config.toml

# set systemd as cgroup driver
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
wget https://github.com/containernetworking/plugins/releases/download/v1.3.0/cni-plugins-linux-amd64-v1.3.0.tgz

# install CNI plugins
mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.3.0.tgz

# restart containerd service
sudo systemctl enable containerd
sudo systemctl restart containerd

# nerdctl tool
wget https://github.com/containerd/nerdctl/releases/download/v1.7.0/nerdctl-1.7.0-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local/bin nerdctl-1.7.0-linux-amd64.tar.gz
```
---
1. install **keadm** both on master and worker  
```
wget https://github.com/kubeedge/kubeedge/releases/download/v1.12.6/keadm-v1.12.6-linux-amd64.tar.gz
tar -zxvf keadm-v1.12.6-linux-amd64.tar.gz
cp keadm-v1.12.6-linux-amd64/keadm/keadm /usr/local/bin/keadm
```
2. Master node ONLY
```
keadm init --advertise-address="master-ip" --profile version=v1.12.6 --kube-config=/home/ubuntu/.kube/config
sudo keadm gettoken --kube-config=/home/ubuntu/.kube/config
```
Uninstall keadm (optional)
> keadm reset --kube-config=$HOME/.kube/config

4. KubeEdge worker node
> keadm join --cloudcore-ipport="master-ip":10000 --kubeedge-version=v1.12.6 --token=$token 
---
### Debugging
Edge node
> journalctl -u edgecore.service-b | tail
