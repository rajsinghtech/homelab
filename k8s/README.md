# Install K3s Ubuntu
## Prepare Node
### Set `MACAddressPolicy=none`
```sh
nano /usr/lib/systemd/network/99-default.link

sudo apt update
sudo apt install nfs-common
```

## Control Plane Node
```sh
ufw disable
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC='--flannel-backend=none --disable-network-policy --disable traefik --disable servicelb --disable-kube-proxy --node-name K8S-STG1.lan --token 12345' sh -

curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET INSTALL_K3S_EXEC='--flannel-backend=none --disable-network-policy --disable traefik --disable servicelb --disable-kube-proxy --cluster-init --tls-san=10.69.200.0' sh - server --cluster-init --tls-san=10.69.200.0

curl -sfL https://get.k3s.io | K3S_TOKEN=K1049e880049237b1dd7f76ebfa2debed1ec6be70eef8d1eb2f9afd17de35002584::server:SECRET INSTALL_K3S_EXEC='--flannel-backend=none --disable-network-policy --disable traefik --disable servicelb --disable-kube-proxy --server https://10.69.200.1:6443 --tls-san=10.69.200.0' sh - server --server https://10.69.200.1:6443 --tls-san=10.69.200.0
```
### Install Cilium
```sh
cilium install --version v1.14.3 --set cluster.name=K8S-STG --set cluster.id=1 --set ipv4NativeRoutingCIDR=10.42.0.0/16 --helm-set k8sServiceHost=K8S-STG1.lan --helm-set k8sServicePort=6443 --helm-set kubeProxyReplacement="strict" --helm-set l2announcements.enabled=true --helm-set l2announcements.leaseDuration="3s" --helm-set l2announcements.leaseRenewDeadline="1s" --helm-set l2announcements.leaseRetryPeriod="500ms" --helm-set devices="{eth0}" --helm-set externalIPs.enabled=true --helm-set hubble.relay.enabled=true --helm-set hubble.ui.enabled=true --helm-set operator.replicas=1

cilium install --version v1.14.3 --set cluster.name=K8S-PROD --set cluster.id=1 --set ipv4NativeRoutingCIDR=10.42.0.0/16 --helm-set k8sServiceHost=K8S-PROD.lan --helm-set k8sServicePort=6443 --helm-set kubeProxyReplacement="strict" --helm-set l2announcements.enabled=true --helm-set l2announcements.leaseDuration="3s" --helm-set l2announcements.leaseRenewDeadline="1s" --helm-set l2announcements.leaseRetryPeriod="500ms" --helm-set devices="{eth0}" --helm-set externalIPs.enabled=true --helm-set hubble.relay.enabled=true --helm-set hubble.ui.enabled=true --helm-set operator.replicas=1

```
### Install Flux
```sh
flux bootstrap github \
  --token-auth \
  --owner=rajsinghtech \
  --repository=homelab \
  --branch=main \
  --path=k8s/clusters/prod \
  --personal    
```

## Worker Nodes
```sh
ufw disable
curl -sfL https://get.k3s.io | K3S_URL=https://K8S-STG1.lan:6443 K3S_TOKEN=K105798aac87939b9f799add02e363a18da09d62b93356853edbf8b1116274d929a::server:12345 INSTALL_K3S_EXEC='--flannel-backend=none --disable-network-policy --disable traefik --disable servicelb --disable-kube-proxy --node-name K8S-STG2.lan' sh -
```
