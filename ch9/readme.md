# create cluster with kind
```bash
kind create cluster --config=kind/config.yaml
```

# delete cluster with kind
```bash
kind delete cluster -n packt-fluxcd-mgmt
```

# start capi control plane controllers
```bash
clusterctl init 
```

# start capi control plane controllers with docker
```bash
clusterctl init --infrastructure docker
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
