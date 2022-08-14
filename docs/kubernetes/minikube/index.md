# Requirements

Homebrew should be installed, if not look [here](https://brew.sh/index_de)

## Packages

Use homebrew to install some of the fundamantels

```bash
brew install podman minikube kubectl k9s octant
```

The installed tools are:

- podman   => docker alternative [website](https://podman.io/)
- minikube => local kunbernetes development environment [website](https://minikube.sigs.k8s.io/docs/start/)
- kubectl  => kubernetes commandline
- k9s      => terminal based kubernetes GUI [website](https://k9scli.io/)
- octant   => electron based kubernetes GUI [website](https://github.com/vmware-tanzu/octant)

The GUI tools are a work in Progress, which one feels better or if everything will be kubectl aliases.

### Prerequisites - Podman

Starting and initializing podman

```bash
# Initialise podman
# minikube wants at least 2 CPUS
# minikube wants at least 2 GB RAM
podman machine init --cpus 4 --memory 8192
# creating the VM
podman machine start
#
# verify
podman info
```

### Prerequisites - minikube

Configure minikube to work with podman. A detailed explanation can be found [here](https://minikube.sigs.k8s.io/docs/drivers/podman/)

```bash
# configure minikube to use podman
minikube config set driver podman
# configure minikube to use containerd as container runtime
minikube config set container-runtime containerd
# configure minikube to use rootless podman
minikube config set rootless true
# start minikube
minikube start
```

### Cluster check

Now a cluster should be starting.
you can check on it with the following commands:

```bash
➜  ~ podman info
host:
  arch: arm64
  buildahVersion: 1.26.1
  cgroupControllers:
...
version:
  APIVersion: 4.1.1
  Built: 1658516809
  BuiltTime: Fri Jul 22 21:06:49 2022
  GitCommit: ""
  GoVersion: go1.18.4
  Os: linux
  OsArch: linux/arm64
  Version: 4.1.1
```

```bash
➜  ~ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

```bash
➜  ~ kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:41193
CoreDNS is running at https://127.0.0.1:41193/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
