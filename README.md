# fluxcd
My adventure into GitOps with fluxcd and my home microk8s cluster.

# Setup Ubuntu server on some shitty piece of sillicon
I went with [Ubuntu server 19.10](https://ubuntu.com/download/server), because why not fuck my shit up!

# Install [microk8s](https://microk8s.io/docs/)
```bash
# Install microk8s
snap install microk8s --classic

# Enable addons
sudo microk8s.enable dns helm storage

# Get kube config and remember to change address to other than localhost
sudo microk8s.kubectl config view --raw
```

## Install/deploy [fluxcd](https://fluxcd.io/) onto microk8s (requires helm, tiller and dns enabled)
Create a Github public repo.
```bash
# Add fluxcd repo to helm
helm repo add fluxcd https://charts.fluxcd.io

# Apply flux custom resource
kubectl apply -f https://raw.githubusercontent.com/fluxcd/flux/helm-0.10.1/deploy-helm/flux-helm-release-crd.yaml

# Install flux helm operator
helm upgrade -i flux \
--set helmOperator.create=true \
--set helmOperator.createCRD=false \
--set git.url=git@github.com:YOURGITHUBUSER/REPO \
--set additionalArgs={--sync-garbage-collection} \ # comment this line if you do not want flux to delete resources
--namespace flux \
fluxcd/flux

# Get flux ssh key and add it to your github repo with allow write access
fluxctl identity --k8s-fwd-ns flux
```