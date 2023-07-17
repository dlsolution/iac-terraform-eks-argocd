# Provision the EKS cluster

```bash
terraform init
terraform plan
terraform apply
```

# Configure kubectl

Run the following command to retrieve the access credentials for your cluster and configure kubectl.

```bash
aws eks --region $(terraform output -raw region) update-kubeconfig --name $(terraform output -raw cluster_name)
```

You can now use `kubectl` to manage your cluster and deploy Kubernetes configurations to it.

Verify the Cluster
```bash
kubectl cluster-info
Kubernetes control plane is running at https://128CA2A0D737317D36E31D0D3A0C366B.gr7.us-east-2.eks.amazonaws.com
CoreDNS is running at https://128CA2A0D737317D36E31D0D3A0C366B.gr7.us-east-2.eks.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

Now verify that all three worker nodes are part of the cluster.
```bash
kubectl get nodes
NAME                                       STATUS   ROLES    AGE     VERSION
ip-10-0-1-50.us-east-1.compute.internal    Ready    <none>   6m20s   v1.24.7-eks-fb459a0
ip-10-0-3-158.us-east-1.compute.internal   Ready    <none>   6m41s   v1.24.7-eks-fb459a0
ip-10-0-3-46.us-east-1.compute.internal    Ready    <none>   6m14s   v1.24.7-eks-fb459a0
```

# Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

# Access The Argo CD API Server

## Service Type Load Balancer

Change the argocd-server service type to LoadBalancer:

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

## Ingress

Follow the [ingress documentation](https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/) on how to configure Argo CD with ingress.

## Port Forwarding
Kubectl port-forwarding can also be used to connect to the API server without exposing the service.

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

The API server can then be accessed using `https://localhost:8080`

> The default username is `admin` and you can get the password from the command below which is stored as a secret in your cluster.

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
