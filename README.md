## Kubernetes policy enforcement with Kyverno

Kyverno is a policy engine for Kubernetes that enforces best practicies and security standards by ensuring Kuberetest resources meet the polices set by the cluster admin.

Policies in this repo:

- Pull images from DOCR only
- Ensure deployments have a `created-by` label
- Prevent images without a tag or with a `latest` tag from being deployed

### Prerequisites

- [DOKS (Digital Ocean Kubernetes) cluster created](https://docs.digitalocean.com/products/kubernetes/quickstart/)
- [DOCR (Digital Ocean Container Registry ) created](https://docs.digitalocean.com/products/container-registry/quickstart/)
- `kubectl` CLI installed
- `docker` CLI installed
- `doctl` CLI installed (For Docker login)
- [`helmsman`](https://github.com/Praqma/helmsman) or any other tool to install Helm charts

Note: Allow DOKS integration with DOCR (DOCR > Settings)

### Build and push sample image

```bash
docker build -t registry.digitalocean.com/<your-docr-registry-name>/date-loop:v1.0.0 registry.digitalocean.com/<your-docr-registry-name>/date-loop:latest .
docker push registry.digitalocean.com/<your-docr-registry-name>/date-loop --all-tags
```

### Install Kyverno and apply policie

```bash
# Helmsman is usueful because it locks down chart versions and protects the resources from accidental deletion

helmsman --apply -f helmsman.yaml
kubectl apply -f policies.yaml -n secure

# Other installation approach
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo update
helm install kyverno kyverno/kyverno --namespace secure --create-namespace
kubectl apply -f policies.yaml
```

### Test policies

```bash
# Random image from Docker Hub (Should fail)
kubectl create deployment nginx --image=nginx:latest

# Deployment without label and uses the latest tag (Should fail)
kubectl apply -f rejected-deployment.yaml

# Good image that fulfills policy requirements (Should deploy successfullyd)
kubectl apply -f good-deployment.yaml
kubectl logs deployment/date-loop -n secure
```

### Demo

[![asciicast](https://asciinema.org/a/cFQ19rnbjFesUjcFNwVBpywz1.svg)](https://asciinema.org/a/cFQ19rnbjFesUjcFNwVBpywz1)

### License

MIT
