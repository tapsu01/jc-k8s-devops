# Install K8S Production environment

**Be careful, follow step-by-step installation bellow:**

> In Document, i use:
>
> - VPS OS: CentOS 7+
> - Local OS: Ubuntu 20.04

### Install List

- [x] Container runtimes: Docker
- [x] Dashboard
- [x] Calico: network plugin
- [x] Traefik: Load balancer
- [ ] Add cluster node
- [ ] Storage

## Container runtimes: Docker

```none
# (Install Docker CE)
## Set up the repository
### Install required packages
$ yum install -y yum-utils device-mapper-persistent-data lvm2
```

```none
## Add the Docker repository
$ yum-config-manager --add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
```

```none
# Install Docker CE
$ yum update -y && yum install -y \
  containerd.io-1.2.13 \
  docker-ce-19.03.11 \
  docker-ce-cli-19.03.11
```

```none
## Create /etc/docker
$ mkdir /etc/docker
```

```none
# Set up the Docker daemon
$ cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
```

```none
$ mkdir -p /etc/systemd/system/docker.service.d
```

```none
# Restart Docker
$ systemctl daemon-reload
$ systemctl restart docker
```

If you want the docker service to start on boot, run the following command:

```none
$ sudo systemctl enable docker
```

## Turnoff Firewall

```none
$ systemctl disable firewalld >/dev/null 2>&1
$ systemctl stop firewalld
```

## Turnoff swap

```none
$ swapoff -a
$ sed -i '/swap/d' /etc/fstab
```

## Installing kubeadm

```none
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
$ sudo sysctl --system
```

```none
$ cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

# Set SELinux in permissive mode (effectively disabling it)
$ sudo setenforce 0
$ sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

$ sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

$ sudo systemctl enable --now kubelet
```

```none
$ systemctl daemon-reload
$ systemctl restart kubelet
```

## Creating a cluster with kubeadm

```none
$ kubeadm init --apiserver-advertise-address={IP_SERVER} --pod-network-cidr=192.168.0.0/16
```

- add calico network addon:

```none
$ kubectl apply -f https://docs.projectcalico.org/v3.16/manifests/calico.yaml
```

## Remote kubernetes from local

> Make sure that you have:
>
> - kubectl [installed](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

[Vietnamese references](https://xuanthulab.net/gioi-thieu-va-cai-dat-kubernetes-cluster.html)

- Copy kubectl config file from server to local computer

```none
$ scp root@{SERVER_IP}:/etc/kubernetes/admin.conf ~/.kube/{CONFIG_FILE_NAME}
```

- For only use in the current session

```none
$ export KUBECONFIG=$HOME/.kube/{CONFIG_FILE_NAME}
# Or
$ export KUBECONFIG=~/.kube/{CONFIG_FILE_NAME}
```

- Use multiple context with kubectl config

```none
$ export KUBECONFIG=~/.kube/config:~/.kube/{CONFIG_FILE_NAME}
$ kubectl config view --flatten > ~/.kube/config_temp
$ mv ~/.kube/config_temp ~/.kube/config
```

- Confirm the change by:

```none
$ kubectl config get-contexts
```

- Switch between contexts

```none
$ kubectl config use-context {CONTEXT_NAME}
```

## Re-name context

- Run command with old name and new name

```none
$ kubectl config rename-context <old-name> <new-name>
```

- Confirm the change by

```none
$ kubectl config get-contexts
```

## Install Dashboard

[Follow link](https://xuanthulab.net/cai-dat-va-su-dung-kubernetes-dashboard.html)

- Generate token for access kubernetes dashboard:

```none
$ kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

## Allow add pod on master node

```none
$ kubectl taint nodes --all node-role.kubernetes.io/master-
```

## [Jetcare project deployment](Jetcare-production-deploy.md)

## Refenrences

1. [Introduction & install - Vietnamese version](https://xuanthulab.net/gioi-thieu-va-cai-dat-kubernetes-cluster.html)
2. [Introduction & install - Kubernetes Home Page](https://kubernetes.io/docs/setup/production-environment/)
3. [K8S - Collect](https://github.com/tapsu01/devops/tree/master/K8s)
