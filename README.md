# metallb-calico
Experimenting with MetalLB and Calico setup (using Minikube). 

Trying to reproduce the following issue [#1603](https://github.com/projectcalico/calico/issues/1603). More context for this problem can be found on the MetalLB repo issue [#114](https://github.com/google/metallb/issues/114#issuecomment-357547646), where the ongoing discussion is occuring. 

A good summary of the issue can also be found on the [official MetalLB documentation](https://metallb.universe.tf/configuration/calico/). 

## The Problem
![Problem](https://mermaidjs.github.io/mermaid-live-editor/#/view/eyJjb2RlIjoiZ3JhcGggQlRcbiAgICBzdWJncmFwaCBcIlwiXG4gICAgICBtZXRhbGxiQVxuICAgICAgY2FsaWNvQVxuICAgIGVuZFxuICAgIHN1YmdyYXBoIFwiXCJcbiAgICAgIG1ldGFsbGJCXG4gICAgICBjYWxpY29CXG4gICAgZW5kXG4gICAgbWV0YWxsYkEoTWV0YWxMQjxicj5zcGVha2VyKS0uIFwiTEIgcm91dGVzPGJyPihkb2Vzbid0IHdvcmspXCIgLi0-cm91dGVyKEJHUCBSb3V0ZXIpXG4gICAgY2FsaWNvQShcIkNhbGljb1wiKS0tIENsdXN0ZXIgcm91dGVzIC0tPnJvdXRlclxuXG4gICAgbWV0YWxsYkIoTWV0YWxMQjxicj5zcGVha2VyKS0uIFwiTEIgcm91dGVzPGJyPihkb2Vzbid0IHdvcmspXCIgLi0-cm91dGVyXG4gICAgY2FsaWNvQihDYWxpY28pLS0gQ2x1c3RlciByb3V0ZXMgLS0-cm91dGVyXG5cbiAgICBzdHlsZSBtZXRhbGxiQSBmaWxsOiNjY2Ysc3Ryb2tlOiNmNjYsc3Ryb2tlLXdpZHRoOjJweCxzdHJva2UtZGFzaGFycmF5OiA1LCA1XG4gICAgc3R5bGUgbWV0YWxsYkIgZmlsbDojY2NmLHN0cm9rZTojZjY2LHN0cm9rZS13aWR0aDoycHgsc3Ryb2tlLWRhc2hhcnJheTogNSwgNVxuIiwibWVybWFpZCI6eyJ0aGVtZSI6ImRlZmF1bHQifX0)

## The Solution
![Solution](https://mermaidjs.github.io/mermaid-live-editor/#/view/eyJjb2RlIjoiZ3JhcGggQlRcbiAgICBzdWJncmFwaCBcIlwiXG4gICAgICBtZXRhbGxiQVxuICAgICAgY2FsaWNvQVxuICAgIGVuZFxuICAgIHN1YmdyYXBoIFwiXCJcbiAgICAgIG1ldGFsbGJCXG4gICAgICBjYWxpY29CXG4gICAgZW5kXG4gICAgbWV0YWxsYkEoTWV0YWxMQjxicj5zcGVha2VyKS0uIFwicGVlcmluZyB0aHJvdWdoIDEyNy4wLjAuMVwiIC4tPmNhbGljb0FcbiAgICBjYWxpY29BKFwiQ2FsaWNvXCIpLS0gQ2x1c3RlciByb3V0ZXMgLS0-cm91dGVyKEJHUCBSb3V0ZXIpXG5cbiAgICBtZXRhbGxiQihNZXRhbExCPGJyPnNwZWFrZXIpLS4gXCJwZWVyaW5nIHRocm91Z2ggMTI3LjAuMC4xXCIgLi0-Y2FsaWNvQlxuICAgIGNhbGljb0IoQ2FsaWNvKS0tIENsdXN0ZXIgcm91dGVzIC0tPnJvdXRlclxuc3R5bGUgbWV0YWxsYkEgZmlsbDojY2NmLHN0cm9rZTojZjY2LHN0cm9rZS13aWR0aDoycHgsc3Ryb2tlLWRhc2hhcnJheTogNSwgNVxuc3R5bGUgbWV0YWxsYkIgZmlsbDojY2NmLHN0cm9rZTojZjY2LHN0cm9rZS13aWR0aDoycHgsc3Ryb2tlLWRhc2hhcnJheTogNSwgNVxuIiwibWVybWFpZCI6eyJ0aGVtZSI6ImRlZmF1bHQifX0)

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
