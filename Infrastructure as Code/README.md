# Pre-Requisites

## Have the following tools installed:

- Git (https://git-scm.com/downloads) (Version: 2.39.2)
- Terraform (https://www.terraform.io/) (Version: v1.4.6)
- Docker (https://www.docker.com/) (Engine: 23.0.5; Server: 4.19.0)
- Azure CLI (https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)

## Accounts

- GitHub Account (https://github.com/)
- DockerHub Account (https://hub.docker.com/)
- Azure Account (Using Student Credit)

<br>
<br>

# The Problem...

We're scaling our infrastructure and we're expected to employ best practices and use the best technologies in order to better maintain it. 

Some things to take into consideration:
- We want to have an easier way to deploy new resources
- We want to have a consistent and reproducible infrastructure
- Employ versioning to our infrastructure is a must
- Remove "works on my machine" and human action errors
- We want to have a declarative way of managing our infra rather than an imperative one

<br>
<br>

# Warmup

Let's go over the basics!

## Objectives:
- Learn the basics of Terraform usage
- Deploy the ToDo List using only Terraform
- Learn the fundamental Terraform commands (plan, apply, validate, fmt, state list)

## Notes:
- Use the [docker-provider](https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs)
- Create a provider.tf file that will declare the above provider
- Create a docker.tf file that will receive the remaining app configuration
- Make sure you also create a docker network where all the parts will live via Terraform
- Create the ws-frontend configuration
- Create the ws-database configuration
- Create the ws-backend configuration

By the end of it, don't forget to run ```terraform destroy```

<br>
<br>

# The Challenge

Let's take advantage of AWS Free Tiers to deploy a compute resource and interact with it, all via Terraform, of course!

- Set up the AWS provider either via Terraform configuration or via environment credentials
- Let's migrate our state file to and AWS S3 (we do need to work as a team, right?)
- Let's use State Locking with AWS Dynamo DB (Secure our state file)
- Create an EC2 configuration in Terraform, apply it!
- We want to SSH into it so let's create a Security Group that allows accessibility to port 22
- Create an SSH Key-Pair that will allow us to connect to our compute instance (Do we need to change anything in our EC2 configuration?)
- Let's test our access to the EC2 instance
- Create a Postgres resource in AWS

