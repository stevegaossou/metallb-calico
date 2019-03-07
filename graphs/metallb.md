### MetalLB 
```
graph BT
    subgraph ""
      metallbA("MetalLB<br>Speaker")
    end
    subgraph ""
      metallbB("MetalLB<br>Speaker")
    end

    subgraph ""
      metallbC("MetalLB<br>Speaker")
    end
    subgraph ""
      metallbD("MetalLB<br>Speaker")
    end

    metallbA-->torA(ToR Router)
    metallbB-->torA

    metallbC-->torB(ToR Router)
    metallbD-->torB
    
    torA-->spine(Spine Router)
    torB-->spine(Spine Router)
```

### MetalLB Calico Problem
```
graph BT
    subgraph ""
      metallbA
      calicoA
    end
    subgraph ""
      metallbB
      calicoB
    end
    metallbA(MetalLB<br>speaker)-. "LB routes<br>(doesn't work)" .->router(BGP Router)
    calicoA("Calico")-- Cluster routes -->router

    metallbB(MetalLB<br>speaker)-. "LB routes<br>(doesn't work)" .->router
    calicoB(Calico)-- Cluster routes -->router

    style metallbA fill:#ccf,stroke:#f66,stroke-width:2px,stroke-dasharray: 5, 5
    style metallbB fill:#ccf,stroke:#f66,stroke-width:2px,stroke-dasharray: 5, 5
```

### MetalLB Calico Solution
```
graph BT
    subgraph ""
      metallbA
      calicoA
    end
    subgraph ""
      metallbB
      calicoB
    end
    metallbA(MetalLB<br>speaker)-. "peering through 127.0.0.1" .->calicoA
    calicoA("Calico")-- Cluster routes -->router(BGP Router)

    metallbB(MetalLB<br>speaker)-. "peering through 127.0.0.1" .->calicoB
    calicoB(Calico)-- Cluster routes -->router

    style metallbA fill:#ccf,stroke:#f66,stroke-width:2px,stroke-dasharray: 5, 5
    style metallbB fill:#ccf,stroke:#f66,stroke-width:2px,stroke-dasharray: 5, 5
```