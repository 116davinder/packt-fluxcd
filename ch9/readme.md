# create cluster with kind
```bash
kind create cluster --config=kind/config.yaml
```

# delete cluster with kind
```bash
kind delete cluster -n packt-fluxcd-mgmt
```

# start capi control plane controllers with docker
```bash
clusterctl init --infrastructure docker
```

# generate the docker provisioned cluster capi yaml
```bash
clusterctl generate cluster packt-dev --flavor development --kubernetes-version v1.29.0 --control-plane-machine-count=1 --worker-machine-count=1 > ch9/capi-clusters/docker/packt-dev.yaml
```

# apply the generated yaml
```bash
$ kubectl apply -f ch9/capi-clusters/docker/packt-dev.yaml
clusterclass.cluster.x-k8s.io/quick-start created
dockerclustertemplate.infrastructure.cluster.x-k8s.io/quick-start-cluster created
kubeadmcontrolplanetemplate.controlplane.cluster.x-k8s.io/quick-start-control-plane created
dockermachinetemplate.infrastructure.cluster.x-k8s.io/quick-start-control-plane created
dockermachinetemplate.infrastructure.cluster.x-k8s.io/quick-start-default-worker-machinetemplate created
dockermachinepooltemplate.infrastructure.cluster.x-k8s.io/quick-start-default-worker-machinepooltemplate created
kubeadmconfigtemplate.bootstrap.cluster.x-k8s.io/quick-start-default-worker-bootstraptemplate created
```

# verify cluster
```bash
$ kubectl get cluster
NAME        CLUSTERCLASS   PHASE         AGE    VERSION
packt-dev   quick-start    Provisioned   108s   v1.29.0
$ clusterctl describe cluster packt-dev
NAME                                                     READY  SEVERITY  REASON                       SINCE  MESSAGE
Cluster/packt-dev                                        True                                          97s
├─ClusterInfrastructure - DockerCluster/packt-dev-665lp  True                                          2m6s
├─ControlPlane - KubeadmControlPlane/packt-dev-8hjgx     True                                          97s
│ └─Machine/packt-dev-8hjgx-pqfc9                        True                                          97s
└─Workers
  ├─MachineDeployment/packt-dev-md-0-jt42m               False  Warning   WaitingForAvailableMachines  2m7s   Minimum availability requires 1 replicas, current 0 available
  │ └─Machine/packt-dev-md-0-jt42m-n9nrc-npvg4           True                                          84s
  └─MachinePool/packt-dev-mp-0-rffx7                     False  Info      WaitingForReplicasReady      31s
    └─Machine/worker-harw1g                              True                                          31s
$ kubectl get kubeadmcontrolplane
NAME              CLUSTER     INITIALIZED   API SERVER AVAILABLE   REPLICAS   READY   UPDATED   UNAVAILABLE   AGE     VERSION
packt-dev-8hjgx   packt-dev   true     
```

# generate kubeconfig for provisioned cluster
```bash
clusterctl get kubeconfig packt-dev > packt-dev.kubeconfig
```

# find API server port on the provisioned cluster
```bash
$ docker inspect --format '{{ (index (index .NetworkSettings.Ports "6443/tcp") 0).HostPort }}' packt-dev-lb
32769
```

# edit the kubeconfig of provisioned cluster
```bash
$ grep server packt-dev.kubeconfig
    #server: https://172.18.0.3:6443
    server: https://localhost:32769
```

# install the calico CNI because nodes stay in unready state
```bash
$ kubectl --kubeconfig=./packt-dev.kubeconfig get nodes
NAME                               STATUS     ROLES           AGE   VERSION
packt-dev-8hjgx-pqfc9              NotReady   control-plane   26m   v1.29.0
packt-dev-md-0-jt42m-n9nrc-npvg4   NotReady   <none>          26m   v1.29.0
packt-dev-worker-harw1g            NotReady   <none>          25m   v1.29.0
```

```bash
$ kubectl --kubeconfig=./packt-dev.kubeconfig create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml
poddisruptionbudget.policy/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
serviceaccount/calico-node created
serviceaccount/calico-cni-plugin created
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
.......
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-cni-plugin created
daemonset.apps/calico-node created
deployment.apps/calico-kube-controllers created
```

```bash
$ kubectl --kubeconfig=./packt-dev.kubeconfig get nodes
NAME                               STATUS   ROLES           AGE   VERSION
packt-dev-8hjgx-pqfc9              Ready    control-plane   40m   v1.29.0
packt-dev-md-0-jt42m-n9nrc-npvg4   Ready    <none>          40m   v1.29.0
packt-dev-worker-harw1g            Ready    <none>          39m   v1.29.0
```

## How to install flux from git repo
```bash
flux bootstrap git \
  --url=ssh://git@github.com/116davinder/packt-fluxcd.git \
  --branch=main \
  --silent \
  --private-key-file=/home/pox/.ssh/id_rsa \
  --path=ch9/flux-cluster/packt-fluxcd \
  --namespace=flux-system
```
