#I'm using [MicroK8s](https://microk8s.io/) in a single node cluster on a Ubuntu 22.04.2 LTS install.

##Install via snap

sudo snap install microk8s --classic
microk8s status --wait-ready

2. Enable plugins

microk8s enable dns ingress cert-manager


1. Enable MetalLB
My subnet is 10.1.1.0/24 and I've excluded 200-254 from DHPC assignment to reserve for MetalLB.

microk8s enable metallb:10.1.1.200-10.1.1.254


3. Add some aliases to my profile

alias kubectl='microk8s kubectl'


4. Getting external DNS working.
a. Add a static route at the router from the K8 subnet to the K8 host.
10.1.199.0/24 10.1.1.10 1


4a. Add my router DNS to K8
cat /etc/nuc.conf
nameserver 10.1.1.1

rg nuc /var/snap/microk8s/current/args/kubelet
--resolv-conf=/etc/nuc.conf



4b Modify the coreDNS configmap
Add Google's DNS server

Either command should work

Update Corefile forward . /etc/resolv.conf to forward . /etc/resolv.conf 8.8.8.8
kubectl get configmap -n kube-system coredns -o yaml > coredns.yaml
EDIT FILE
kubectl apply -f coredns.yaml

OR add it by hand

microk8s kubectl -n kube-system edit configmap/coredns

DNS Troubleshooting

#Check for DNS Errors
kubectl logs --namespace=kube-system -l k8s-app=kube-dns


#nslookup 
kubectl exec dnsutils -it -- nslookup google.com


Drop into a busybox session on the cluster
kubectl run curl --image=radial/busyboxplus:curl -i --tty
(Can then run nslookup from inside the cluster.)
Reattach
kubectl attach -it curl
