# metallb-calico
Experimenting with MetalLB and Calico setup (using Minikube). 

Trying to reproduce the following issue [#1603](https://github.com/projectcalico/calico/issues/1603). More context for this problem can be found on the MetalLB repo issue [#114](https://github.com/google/metallb/issues/114#issuecomment-357547646), where the ongoing discussion is occuring. 

A good summary of the issue can also be found on the [official MetalLB documentation](https://metallb.universe.tf/configuration/calico/). 

## Enviroment 
- Calico version: `v3.5.2`
- Orchestrator version: `Kubernetes v1.13.3 using kubeadm`
- Operating System and version: `minikube v0.34.1 on Darwin 18.2.0`

## Setup
The following setup is based on the [MetalLB Minikube tutorial](https://metallb.universe.tf/tutorial/minikube/). 

#### Create Minikube Cluster 
- `minikube start`

#### Install router and open browser to UI 
- `kube apply -f 00-test-bgp-router.v0.7.3.yaml`
- `minikube service -n metallb-system test-bgp-router-ui`
- `kube get pods --all-namespaces --watch`

#### Install Calico and tail logs and examine peering
- `kube apply -f 01-calico.yaml`
- `kube exec <calico-node-id> -it sh --namespace kube-system`
```
    # Download the matching version
    curl -O -L  https://github.com/projectcalico/calicoctl/releases/download/v3.5.0/calicoctl

    # Make it executable
    chmod +x calicoctl

    # If you like you can move it to a valid PATH
    mv calicoctl /usr/local/bin/calicoctl

    # Exmaine peering
    sudo calico node status
```

- `kube logs -f <calico-node-id> -n kube-system`

#### Configure Calico to Peer with Router and loopback
`calicoctl apply -f 02-bgppeer-routers.yaml`
`calicoctl apply -f 03-bgppeer-metallb.yaml`

#### Install MetalLB and Configure 
- `kube apply -f 04-metallb.v0.7.3.yaml`
- `kube apply -f 05-metallb-config.yaml`
- `kube logs -f <speaker-id> -n metallb-system`

#### Install new service of type LoadBalancer 
- `kube apply -f 06-service-loadbalancer.yaml`
