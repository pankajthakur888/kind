# Kind High Availability (HA) Cluster Setup

## Installing Kind

### On Linux:
```sh
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.26.0/kind-$(uname)-amd64
# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.26.0/kind-$(uname)-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

### On macOS via Homebrew:
```sh
brew install kind
```

### On macOS via MacPorts:
```sh
sudo port selfupdate && sudo port install kind
```

### On macOS via Bash:
```sh
# For Intel Macs
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.26.0/kind-darwin-amd64
# For M1 / ARM Macs
[ $(uname -m) = arm64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.26.0/kind-darwin-arm64
chmod +x ./kind
mv ./kind /some-dir-in-your-PATH/kind
```

### On Windows:
```sh
curl.exe -Lo kind-windows-amd64.exe https://kind.sigs.k8s.io/dl/v0.26.0/kind-windows-amd64
Move-Item .\kind-windows-amd64.exe c:\some-dir-in-your-PATH\kind.exe

# OR via Chocolatey
choco install kind
```

## Overview
This guide provides steps to set up a highly available (HA) Kind cluster with 3 control-plane nodes and 3 worker nodes. The configuration has been optimized by removing duplicate mounts and ensuring only essential settings are included.

## Corrected Kind Configuration
Save the following content as `kind-ha-network-config.yaml`:

```yaml
# Kind cluster with 3 control-plane nodes and 3 worker nodes
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: cluster-kind-ha-net
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 6443
        hostPort: 6443
        protocol: TCP
  - role: control-plane
  - role: control-plane
  - role: worker
  - role: worker
  - role: worker

networking:
  disableDefaultCNI: false
  podSubnet: "10.244.0.0/16"
  serviceSubnet: "10.96.0.0/12"

kubeadmConfigPatches:
  - |
    apiVersion: kubeadm.k8s.io/v1beta3
    kind: ClusterConfiguration
    metadata:
      name: config
    controllerManager:
      extraArgs:
        bind-address: 0.0.0.0
    scheduler:
      extraArgs:
        bind-address: 0.0.0.0
    etcd:
      local:
        extraArgs:
          listen-metrics-urls: http://0.0.0.0:2381

  - |
    apiVersion: kubeadm.k8s.io/v1beta3
    kind: KubeProxyConfiguration
    mode: "ipvs"
```

## What Changed?
✅ Removed duplicate `/lib/modules` mount
✅ Ensured only essential `extraPortMappings` are included
✅ Retained all required networking settings


## Creating the Kind Cluster
Run the following command to create the Kind cluster:

```sh
kind create cluster --name cluster-kind-ha-net --config kind-ha-network-config.yaml --retain
```

This should resolve any mount conflicts and successfully create the HA cluster. 🚀

## Verifying the Cluster
After the cluster is created, verify that all nodes are running:

```sh
kubectl get nodes
```

You should see output listing all 6 nodes (3 control-plane and 3 worker nodes) in the `Ready` state.

## Using Kind
To create a default cluster:
```sh
kind create cluster
```

To delete your cluster:
```sh
kind delete cluster
```

To create a cluster from Kubernetes source:
```sh
# Add your Kubernetes source cluster creation steps here
```

## Next Steps
- Deploy a CNI plugin (e.g., Calico or Flannel) if needed.
- Deploy workloads and test HA functionality.
- Configure monitoring and logging as required.

Enjoy your HA Kind cluster! 🎉

