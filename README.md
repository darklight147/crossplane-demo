# Getting started

## Preqrequisites

1. A kubernetes Cluster (Tested with version 1.25.4)
2. an IAM aws user
3. Helm 3 installed
4. ArgoCD (optional)

## Installation

1. Clone this repo
2. Install the crossplane helm chart
3. Install your first provider
4. Create a secret with the aws credentials
5. Install the helm chart

### Install the crossplane helm chart

```bash
helm repo add \
crossplane-stable https://charts.crossplane.io/stable
helm repo update

helm install crossplane \
crossplane-stable/crossplane \
--namespace crossplane-system \
--create-namespace
```

### Create a secret with the aws credentials

```bash
kubectl create secret generic aws-secret --from-file creds=$HOME/.aws/credentials --namespace crossplane-system
```

### Install your first provider

```bash
cat <<EOF | kubectl apply -f -
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-aws-s3
spec:
  package: xpkg.upbound.io/upbound/provider-aws-s3:v0.40.0
EOF
```

### Adding secret to the provider

```bash
cat <<EOF | kubectl apply -f -
apiVersion: aws.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: aws-secret
      key: creds
EOF
```

### Create your first bucket

```bash
bucket=$(echo "crossplane-bucket-"$(head -n 4096 /dev/urandom | openssl sha1 | tail -c 10))
region="us-east-1"

cat <<EOF | kubectl apply -f -
apiVersion: s3.aws.upbound.io/v1beta1
kind: Bucket
metadata:
  name: $bucket
spec:
  forProvider:
    region: $region
  providerConfigRef:
    name: default
EOF
```

### Great, now you have a bucket

```bash
kubectl get buckets
```

### Debug

```bash
kubectl get providers
kubectl get managed
```
