# metallb-calico
Experimenting with MetalLB and Calico setup (using Minikube). 

Trying to reproduce the following issue [#1603](https://github.com/projectcalico/calico/issues/1603). More context for this problem can be found on the MetalLB repo issue [#114](https://github.com/google/metallb/issues/114#issuecomment-357547646), where the ongoing discussion is occuring. 

A good summary of the issue can also be found on the [official MetalLB documentation](https://metallb.universe.tf/configuration/calico/). 

Below is a quick summary of the integration issue between Calico & MetalLB. 

## MetalLB 
A typical setup using MetalLB might look something like: 

![screen shot 2019-03-06 at 4 28 33 pm](https://user-images.githubusercontent.com/18451894/53923428-2af64780-402d-11e9-8b1d-985a87486829.png)


## The Problem
The problem occurs when each host also has Calico node installed. This means that both MetalLB and Calico will attempt to peer an external Router. 

![screen shot 2019-03-06 at 4 27 37 pm](https://user-images.githubusercontent.com/18451894/53923371-f5515e80-402c-11e9-9239-5b6d7e8303b3.png)

## The Solution
The most generic solution proposed on the [MetalLB ticket #114](https://github.com/google/metallb/issues/114#issuecomment-357547646) recomends having Calico peer with the external Router and MetalLB peer with Calico on the host. This can be done using the loopback IP (127.0.0.1). 

![screen shot 2019-03-06 at 4 37 38 pm](https://user-images.githubusercontent.com/18451894/53923751-2f6f3000-402e-11e9-920d-0c9b677efc28.png)

The only obstacle with this solution appears to be an issue where Calico node constantly attempts to peer with 127.0.0.1. If MetalLB is not ready then this leads to repeated connection errors. This causes Calico node to enter error backoff period, during which no real peering can occur. 

The goal of this experiement was to verify that this error backoff happens. 

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
