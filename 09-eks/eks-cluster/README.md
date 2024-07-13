# EKS Configuration

## Provision the EKS Cluster

#### Initialize the Terraform Resources
```bash
terraform init
```

#### Plan the Terraform Resources
```bash
terraform plan
```

#### Apply the Terraform Resources
```bash
terraform apply
```

#### Configure kubectl

After provisioning your cluster, you need to configure kubectl to interact with it.

```bash
aws eks --region $(terraform output -raw region) update-kubeconfig \
    --name $(terraform output -raw cluster_name)
```

#### Verify the cluster
```bash
kubectl cluster-info
```
