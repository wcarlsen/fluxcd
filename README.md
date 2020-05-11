# fluxcd
My adventure into GitOps with fluxcd and my home microk8s cluster.

# Setup Ubuntu server on some shitty piece of sillicon
I went with [Ubuntu server 19.10](https://ubuntu.com/download/server), because why not fuck my shit up!

# Install [microk8s](https://microk8s.io/docs/)
```bash
# Install microk8s
snap install microk8s --classic

# Enable addons
sudo microk8s.enable dns storage metrics-server

# Get kube config and remember to change address to other than localhost
sudo microk8s.kubectl config view --raw
```

## Install/deploy [fluxcd](https://fluxcd.io/) onto microk8s (requires helm v3 and dns enabled)
Create a Github public repo.
```bash
# Create flux namespace
kubectl create ns flux

# Add fluxcd repo to helm
helm repo add fluxcd https://charts.fluxcd.io

# Apply flux custom resource
kubectl apply -f https://raw.githubusercontent.com/fluxcd/flux/helm-0.10.1/deploy-helm/flux-helm-release-crd.yaml

# Install flux
helm upgrade -i flux fluxcd/flux --wait \
--namespace flux \
--set registry.pollInterval=1m \
--set git.pollInterval=1m \
--set git.url=git@github.com:wcarlsen/fluxcd \
--set syncGarbageCollection.enabled=true \
--set additionalArgs={--connect=ws://fluxcloud}

# Get flux ssh key and add it to your github repo with allow write access
fluxctl identity --k8s-fwd-ns flux

# Install helmRelease CRD
kubectl apply -f https://raw.githubusercontent.com/fluxcd/helm-operator/helm-v3-dev/deploy/flux-helm-release-crd.yaml

# Install helm-operator
# Note that you can find the latest image tags here https://hub.docker.com/repository/docker/fluxcd/helm-operator-prerelease/tags?page=1&ordering=last_updated
# Also note that zsh might be a bitch, so switch to bash
helm upgrade -i helm-operator fluxcd/helm-operator --wait \
--namespace flux \
--set git.ssh.secretName=flux-git-deploy \
--set git.pollInterval=1m \
--set chartsSyncInterval=1m \
--set configureRepositories.enable=true \
--set configureRepositories.repositories[0].name=stable \
--set configureRepositories.repositories[0].url=https://kubernetes-charts.storage.googleapis.com \
--set configureRepositories.repositories[1].name=loki \
--set configureRepositories.repositories[1].url=https://grafana.github.io/loki/charts \
--set configureRepositories.repositories[2].name=incubator\
--set configureRepositories.repositories[2].url=http://storage.googleapis.com/kubernetes-charts-incubator \
--set extraEnvs[0].name=HELM_VERSION \
--set extraEnvs[0].value=v3 \
--set image.repository=docker.io/fluxcd/helm-operator-prerelease \
--set image.tag=helm-v3-dev-ca9c8ba0
```
