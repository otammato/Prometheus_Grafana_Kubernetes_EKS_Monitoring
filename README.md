# Prometheus_Grafana_Kubernetes_EKS_Monitoring

https://docs.aws.amazon.com/eks/latest/userguide/prometheus.html

https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html

## Use a Terraform template to launch a Kubernetes cluster on AWS EKS

1. Clone this official hashicorp repository with terraform templates:


```
git clone https://github.com/hashicorp/learn-terraform-provision-eks-cluster
```

This example repository contains configuration to provision a VPC, security groups, and an EKS cluster with the following architecture:

<img width="711" alt="Screenshot 2023-05-28 at 17 53 54" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/9ac1f30e-7b2c-4b7c-bcca-95de95b03e04">

2. Launch the terraform template to create infrastructure

```
terraform init
terraform validate
terraform apply
```

<img width="711" alt="Screenshot 2023-05-28 at 17 53 54" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/05c90361-fa22-4b15-8f28-0071bc700691">

> If you use AWS Cloud9 as an IDE you also have to disallow AWS Managed Temporary Credentials
> Go to Cloud9 > Preferences > AWS Settings > AWS Managed Temporary Credentials and turn it off.
> Store your permanent AWS access credentials in the environment. Use ```aws configure``` command.
> <img width="279" alt="Screenshot 2023-05-28 at 14 14 28" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/34f3028e-3fed-4baf-b2af-b1180ea5e5b5">

<img width="584" alt="Screenshot 2023-05-28 at 18 30 32" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/9e0c2e9f-1fba-4496-b28f-798ea60070f2">


