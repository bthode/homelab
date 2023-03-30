#I'm using [MicroK8s](https://microk8s.io/) in a single node cluster on a Ubuntu 22.04.2 LTS install.

## Install via snap
```
sudo snap install microk8s --classic
microk8s status --wait-ready
```

## Enable plugins
```
microk8s enable dns ingress cert-manager registry
```

#### Enable MetalLB
My subnet is 10.1.1.0/24 and I've excluded 200-254 from DHPC assignment to reserve for MetalLB.
```
microk8s enable metallb:10.1.1.200-10.1.1.254
```

### Getting external DNS working.
1. Add a static route at the router from the K8 subnet to the K8 host.
10.1.199.0/24 10.1.1.10 1


2. Add my router DNS to K8
cat /etc/nuc.conf
nameserver 10.1.1.1

rg nuc /var/snap/microk8s/current/args/kubelet
--resolv-conf=/etc/nuc.conf

3. Modify the coreDNS configmap
Add Google's DNS server

Either command should work

Update Corefile forward . /etc/resolv.conf to forward . /etc/resolv.conf 8.8.8.8
kubectl get configmap -n kube-system coredns -o yaml > coredns.yaml
EDIT FILE
kubectl apply -f coredns.yaml

OR add it by hand

kubectl -n kube-system edit configmap/coredns

DNS Troubleshooting

## Trust the local registry
sudo mkdir -p /var/snap/microk8s/current/args/certs.d/10.1.1.10:32000
sudo touch /var/snap/microk8s/current/args/certs.d/10.1.1.10:32000/hosts.toml

# /var/snap/microk8s/current/args/certs.d/10.1.1.10:32000/hosts.toml
server = "http://10.1.1.10:32000"

[host."http://10.1.1.10:32000"]
capabilities = ["pull", "resolve"]

## Then restart MicroK8s
microk8s stop
microk8s start

## To test deploying from the private registry (Update image accordingly)
microk8s kubectl create deployment nginx --image=10.1.1.1:32000/mynginx:latest

## Check for DNS Errors
kubectl logs --namespace=kube-system -l k8s-app=kube-dns

# Test resolution from a pod
kubectl exec dnsutils -it -- nslookup google.com

## Drop into a busybox session on the cluster
kubectl run curl --image=radial/busyboxplus:curl -i --tty
(Can then run nslookup from inside the cluster.)

### Reattach
kubectl attach -it curl
