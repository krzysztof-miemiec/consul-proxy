# Consul Proxy

This Helm chart can be used to expose an external service (i.e. AWS RDS) to Consul Service Mesh.

## My use case
I wanted to use Vault to manage access to database located in a different VPC, on a different AWS account. I can't bridge two-VPCs (IP collisions). I also don't want to expose my database to the Internet. Tunelling with bastion instance is also not an option for me.

To accomplish the task, I created a Deployment with Envoy, which passes all TCP traffic to a local port. Then I simply added consul-connect annotations to the pod.

## Usage with Terraform

I deploy everything with Terraform, so here's how it looks:

```tf
locals {
  postgres_proxy_values = {
    name        = "postgres-rds"
    host        = "your-postgres-hostname.us-east-1.rds.amazonaws.com"
    port        = "5432"
    exposedPort = "5432"
    tlsEnabled  = false
  }
}

resource "kubernetes_namespace" "postgres" {
  metadata {
    name = "postgres-proxy"
  }
}


resource "helm_release" "postgres" {
  source = "./modules/helm-release"

  name      = "postgres-proxy"
  namespace = kubernetes_namespace.postgres.metadata[0].name
  chart     = "${path.module}/charts/consul-proxy"
  wait      = true

  values = [yamlencode(local.postgres_proxy_values)]
}
```