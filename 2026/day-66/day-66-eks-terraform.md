Task 1: Project Setup

1) Create a new project directory with proper file structure:

        terraform-eks/
          providers.tf        # Provider and backend config
          vpc.tf              # VPC module call
          eks.tf              # EKS module call
          variables.tf        # All input variables
          outputs.tf          # Cluster outputs
          terraform.tfvars    # Variable values

In `providers.tf`:

1) Pin the AWS provider to `~> 5.0`

        aws = {
              source  = "hashicorp/aws"
              version = "~> 5.0"
            }

2) Pin the Kubernetes provider (you will need it later)
        
        kubernetes = {
              source  = "hashicorp/kubernetes"
              version = "~> 2.0"
            }

3) Set your region

        provider "aws" {
          region = var.region
        }

`providers.tf`

        terraform {
          required_version = ">= 1.5.0"
        
          required_providers {
            aws = {
              source  = "hashicorp/aws"
              version = "~> 5.0"
            }
        
            kubernetes = {
              source  = "hashicorp/kubernetes"
              version = "~> 2.0"
            }
          }
        }
        
        provider "aws" {
          region = var.region
        }

In `variables.tf`, define:

* `region` (string)
* `cluster_name` (string, default: `"terraweek-eks"`)
* `cluster_version` (string, default: `"1.31"`)
* `node_instance_type` (string, default: `"t3.micro"`)
* `node_desired_count` (number, default: `2`)
* `vpc_cidr` (string, default: `"10.0.0.0/16"`)

`variables.tf`

        variable "region" {
          description = "AWS region for EKS infrastructure"
          type        = string
        }
        
        variable "cluster_name" {
          description = "Name of the EKS cluster"
          type        = string
          default     = "terraweek-eks"
        }
        
        variable "cluster_version" {
          description = "Kubernetes version for EKS"
          type        = string
          default     = "1.31"
        }
        
        variable "node_instance_type" {
          description = "EC2 instance type for EKS worker nodes"
          type        = string
          default     = "t3.micro"
        }
        
        variable "node_desired_count" {
          description = "Desired number of worker nodes"
          type        = number
          default     = 2
        }
        
        variable "vpc_cidr" {
          description = "CIDR block for the VPC"
          type        = string
          default     = "10.0.0.0/16"
        }

---

Task 2: Create the VPC with Registry Module

EKS requires a VPC with both public and private subnets across multiple availability zones.

In `vpc.tf`, use the `terraform-aws-modules/vpc/aws` module:

1) CIDR: `var.vpc_cidr`

2) At least 2 availability zones

3) 2 public subnets and 2 private subnets

4) Enable NAT gateway (single NAT to save cost): `enable_nat_gateway = true`, `single_nat_gateway = true`

5) Enable DNS hostnames: `enable_dns_hostnames = true`

6) Add the required EKS tags on subnets:

        public_subnet_tags = {
          "kubernetes.io/role/elb" = 1
        }
        
        private_subnet_tags = {
          "kubernetes.io/role/internal-elb" = 1
        }

`vpc.tf`

        module "vpc" {
          source  = "terraform-aws-modules/vpc/aws"
          version = "~> 5.0"
        
          name = "${var.cluster_name}-vpc"
          cidr = var.vpc_cidr
        
          azs = ["us-west-2a", "us-west-2b"]
        
          public_subnets = [
            "10.0.1.0/24",
            "10.0.2.0/24"
          ]
        
          private_subnets = [
            "10.0.11.0/24",
            "10.0.12.0/24"
          ]
        
          enable_nat_gateway   = true
          single_nat_gateway   = true
          enable_dns_hostnames = true
          enable_dns_support   = true
        
          public_subnet_tags = {
            "kubernetes.io/role/elb" = "1"
          }
        
          private_subnet_tags = {
            "kubernetes.io/role/internal-elb" = "1"
          }
        
          tags = {
            Environment = "dev"
            ManagedBy   = "Terraform"
            Project     = var.cluster_name
          }
        }

Run `terraform init` and `terraform plan` to verify the VPC config before moving on.

- terraform init

        Initializing the backend...
        
        Initializing modules...
        
        Initializing provider plugins...
        - Reusing previous version of hashicorp/aws from the dependency lock file
        - Reusing previous version of hashicorp/kubernetes from the dependency lock file
        - Using previously-installed hashicorp/aws v5.100.0
        - Using previously-installed hashicorp/kubernetes v2.38.0
        Terraform has been successfully initialized!

- terraform plan

        Plan: 19 to add, 0 to change, 0 to destroy.

*Document*: Why does EKS need both public and private subnets? What do the subnet tags do?

- Public subnets

        These have a route to an Internet Gateway. They can host internet-facing load balancers that receive traffic from users.

- Private subnets

        These are intended for EKS worker nodes and application workloads. A NAT Gateway can provide outbound internet access without giving those nodes direct inbound internet access.

---

Task 3: Create the EKS Cluster with Registry Module

In `eks.tf`, use the `terraform-aws-modules/eks/aws` module:

        module "eks" {
          source  = "terraform-aws-modules/eks/aws"
          version = "~> 20.0"
        
          cluster_name    = var.cluster_name
          cluster_version = var.cluster_version
        
          vpc_id     = module.vpc.vpc_id
          subnet_ids = module.vpc.private_subnets
        
          cluster_endpoint_public_access = true
        
          eks_managed_node_groups = {
            terraweek_nodes = {
              ami_type       = "AL2_x86_64"
              instance_types = [var.node_instance_type]
        
              min_size     = 1
              max_size     = 3
              desired_size = var.node_desired_count
            }
          }
        
          tags = {
            Environment = "dev"
            Project     = "TerraWeek"
            ManagedBy   = "Terraform"
          }
        }
        
Run:

- terraform init      # Download EKS module and its dependencies
        
        Initializing the backend...
        
        Initializing modules...
        Downloading registry.terraform.io/terraform-aws-modules/eks/aws 20.37.2 for eks...
        - eks in .terraform\modules\eks
        - eks.fargate_profile in .terraform\modules\eks\modules\fargate-profile
        Downloading registry.terraform.io/terraform-aws-modules/kms/aws 2.1.0 for eks.kms...
        - eks.kms in .terraform\modules\eks.kms
        - eks.eks_managed_node_group in .terraform\modules\eks\modules\eks-managed-node-group
        - eks.self_managed_node_group in .terraform\modules\eks\modules\self-managed-node-group
        - eks.eks_managed_node_group.user_data in .terraform\modules\eks\modules\_user_data
        - eks.self_managed_node_group.user_data in .terraform\modules\eks\modules\_user_data
        
        Initializing provider plugins...
        - Reusing previous version of hashicorp/kubernetes from the dependency lock file
        - Finding hashicorp/time versions matching ">= 0.9.0"...
        - Finding hashicorp/tls versions matching ">= 3.0.0"...
        - Finding hashicorp/cloudinit versions matching ">= 2.0.0"...
        - Finding hashicorp/null versions matching ">= 3.0.0"...
        - Reusing previous version of hashicorp/aws from the dependency lock file
        - Installing hashicorp/time v0.14.2...
        - Installed hashicorp/time v0.14.2 (signed by HashiCorp)
        - Installing hashicorp/tls v4.4.1...
        - Installed hashicorp/tls v4.4.1 (signed by HashiCorp)
        - Installing hashicorp/cloudinit v2.4.1...
        - Installed hashicorp/cloudinit v2.4.1 (signed by HashiCorp)
        - Installing hashicorp/null v3.3.2...
        - Installed hashicorp/null v3.3.2 (signed by HashiCorp)
        - Using previously-installed hashicorp/aws v5.100.0
        - Using previously-installed hashicorp/kubernetes v2.38.0
        
        Terraform has made some changes to the provider dependency selections recorded
        in the .terraform.lock.hcl file. Review those changes and commit them to your
        version control system if they represent changes you intended to make.
        
        Terraform has been successfully initialized!

- terraform plan      # Review -- this will create 30+ resources

        Plan: 55 to add, 0 to change, 0 to destroy.

Review the plan carefully before applying. You should see: EKS cluster, IAM roles, node group, security groups, and more.

---

Task 4: Apply and Connect kubectl

1) Apply the config:

- terraform apply

        Apply complete! Resources: 55 added, 0 changed, 0 destroyed.
        
        Outputs:
        
        cluster_endpoint = "https://B3F787D6D8C794E03E56C59BCCBC587D.gr7.us-west-2.eks.amazonaws.com"
        cluster_name = "terraweek-eks"
        cluster_region = "us-west-2"

This will take 10-15 minutes. EKS cluster creation is slow -- be patient.

2) Add outputs in `outputs.tf`:

        output "cluster_name" {
          value = module.eks.cluster_name
        }
        
        output "cluster_endpoint" {
          value = module.eks.cluster_endpoint
        }
        
        output "cluster_region" {
          value = var.region
        }

3) Update your kubeconfig:

        aws eks update-kubeconfig --name terraweek-eks --region <your-region>

- aws eks update-kubeconfig --name terraweek-eks --region us-west-2 

        Added new context arn:aws:eks:us-west-2:611622961349:cluster/terraweek-eks to C:\Users\gtmew\.kube\config

4) Verify:

- kubectl get nodes
        
        NAME                                       STATUS   ROLES    AGE   VERSION
        ip-10-0-11-18.us-west-2.compute.internal   Ready    <none>   12m   v1.31.13-eks-ecaa3a6
        ip-10-0-12-17.us-west-2.compute.internal   Ready    <none>   11m   v1.31.13-eks-ecaa3a6

- kubectl get pods -A

        NAMESPACE     NAME                     READY   STATUS    RESTARTS   AGE
        kube-system   aws-node-62mcm           2/2     Running   0          12m
        kube-system   aws-node-gswph           2/2     Running   0          12m
        kube-system   coredns-77c675c6-l4w4j   1/1     Running   0          16m
        kube-system   coredns-77c675c6-mm4fc   1/1     Running   0          16m
        kube-system   kube-proxy-jgq7g         1/1     Running   0          12m
        kube-system   kube-proxy-zxnhc         1/1     Running   0          12m

- kubectl cluster-info

        Kubernetes control plane is running at https://B3F787D6D8C794E03E56C59BCCBC587D.gr7.us-west-2.eks.amazonaws.com
        CoreDNS is running at https://B3F787D6D8C794E03E56C59BCCBC587D.gr7.us-west-2.eks.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

        To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

Verify: Do you see 2 nodes in `Ready` state? Can you see the kube-system pods running?

        Yess!

---

Task 5: Deploy a Workload on the Cluster

Your Terraform-provisioned cluster is live. Deploy something on it.

1) Create a file `k8s/nginx-deployment.yaml`:

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: nginx-terraweek
          labels:
            app: nginx
        spec:
          replicas: 3
          selector:
            matchLabels:
              app: nginx
          template:
            metadata:
              labels:
                app: nginx
            spec:
              containers:
              - name: nginx
                image: nginx:latest
                ports:
                - containerPort: 80
        ---
        apiVersion: v1
        kind: Service
        metadata:
          name: nginx-service
        spec:
          type: LoadBalancer
          selector:
            app: nginx
          ports:
          - port: 80
            targetPort: 80

2) Apply:

- kubectl apply -f k8s/nginx-deployment.yaml

        deployment.apps/nginx-terraweek created
        service/nginx-service created

3) Wait for the LoadBalancer to get an external IP:

- kubectl get svc nginx-service -w

        NAME            TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)        AGE
        nginx-service   LoadBalancer   172.20.195.148   af7ebd7af2be64a8397a800216dce38a-1875114073.us-west-2.elb.amazonaws.com   80:31665/TCP   32s

4) Access the Nginx page via the LoadBalancer URL

        http://af7ebd7af2be64a8397a800216dce38a-1875114073.us-west-2.elb.amazonaws.com

5) Verify the full picture:

- kubectl get nodes
        
        NAME                                       STATUS   ROLES    AGE   VERSION
        ip-10-0-11-18.us-west-2.compute.internal   Ready    <none>   18m   v1.31.13-eks-ecaa3a6
        ip-10-0-12-17.us-west-2.compute.internal   Ready    <none>   18m   v1.31.13-eks-ecaa3a6

- kubectl get deployments
        
        NAME              READY   UP-TO-DATE   AVAILABLE   AGE
        nginx-terraweek   2/3     3            2           2m40s

- kubectl get pods
        
        NAME                               READY   STATUS    RESTARTS   AGE
        nginx-terraweek-54b9c68f67-bxk9x   1/1     Running   0          3m7s
        nginx-terraweek-54b9c68f67-cckf9   1/1     Running   0          3m7s
        nginx-terraweek-54b9c68f67-rfdvp   0/1     Pending   0          3m7s

- kubectl get svc

        NAME            TYPE           CLUSTER-IP       EXTERNAL-IP                   PORT(S)        AGE
        kubernetes      ClusterIP      172.20.0.1       <none>                   443/TCP        25m
        nginx-service   LoadBalancer   172.20.195.148   af7ebd7af2be64a8397a800216dce38a-1875114073.us-west-2.elb.amazonaws.com   80:31665/TCP   3m22s

Verify: Can you access the Nginx welcome page through the LoadBalancer URL?

        Yes, I can.

---

Task 6: Destroy Everything

This is the most important step. EKS clusters cost money. Clean up completely.

1) First, remove the Kubernetes resources (so the AWS LoadBalancer gets deleted):

- kubectl delete -f k8s/nginx-deployment.yaml
        
        deployment.apps "nginx-terraweek" deleted from default namespace
        service "nginx-service" deleted from default namespace

2) Wait for the LoadBalancer to be fully removed (check EC2 > Load Balancers in AWS console)

        Yup, they are gone.

3) Destroy all Terraform resources:

- terraform destroy

        Destroy complete! Resources: 57 destroyed.

This will take 10-15 minutes.

4) Verify in the AWS console:

* EKS clusters: empty

        terraweek-eks is gone

* EC2 instances: no node group instances

        No lab worker nodes remain

* VPC: the terraweek VPC should be gone

        The terraweek-eks-vpc VPC is gone

* NAT Gateways: deleted

        The lab NAT Gateway is deleted

* Elastic IPs: released

        No lab Elastic IP remains allocated

Verify: Is your AWS account completely clean? No leftover resources?

        Yes
