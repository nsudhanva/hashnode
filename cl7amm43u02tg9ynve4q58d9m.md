## Install Kubeflow on Digital Ocean Managed Kubernetes (with Terraform)

Here are the steps to deploy Kubeflow on Digital Ocean Managed Kubernetes for your Machine Learning Workflows. Terraform configuration files can be found here: [https://github.com/nsudhanva/do-k8s-challenge](https://github.com/nsudhanva/do-k8s-challenge)

Contents
--------

*   [Getting Started](https://sudhanva-narayana.ghost.io/#getting-started)
    *   [Installing `kubectl`](https://sudhanva-narayana.ghost.io/#installing-kubectl)
    *   [Installing Terraform](https://sudhanva-narayana.ghost.io/#installing-terraform)
    *   [Installing `doctl`](https://sudhanva-narayana.ghost.io/#installing-doctl)
    *   [Installing `kfctl`](https://sudhanva-narayana.ghost.io/#installing-kfctl)
*   [Creating a Kubernetes Cluster on Digital Ocean using Terraform](https://sudhanva-narayana.ghost.io/#creating-a-kubernetes-cluster-on-digital-ocean-using-terraform)
    *   [Terraform configuration and setup](https://sudhanva-narayana.ghost.io/#terraform-configuration-and-setup)
    *   [Creating the cluster](https://sudhanva-narayana.ghost.io/#creating-the-cluster)
    *   [Setting up `.kubeconfig`](https://sudhanva-narayana.ghost.io/#setting-up-kubeconfig)
    *   [Verifying cluster creation](https://sudhanva-narayana.ghost.io/#verifying-cluster-creation)
    *   [Fixing a few bottlenecks before proceeding](https://sudhanva-narayana.ghost.io/#fixing-a-few-bottlenecks-before-proceeding)
*   [Installing Kubeflow](https://sudhanva-narayana.ghost.io/#installing-kubeflow)
    *   [Installing on Digital Ocean Kubernetes](https://sudhanva-narayana.ghost.io/#installing-on-digital-ocean-kubernetes)
    *   [Accessing the dashboard](https://sudhanva-narayana.ghost.io/#accessing-the-dashboard)
    *   [Attaching a Load Balancer](https://sudhanva-narayana.ghost.io/#attaching-a-loadbalancer)
    *   [Enabling autoscaling on Digital Ocean Kubernetes](https://sudhanva-narayana.ghost.io/#enabling-autoscaling-on-digital-ocean-kubernetes)

Getting Started
---------------

### Installing kubectl

`kubectl` is the command line tool that's used to interact with Kubernetes cluster and configure things in and around Kubernetes. The installation instructions can be found here: [https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)

Depending on your operating system, install the relevant one.

### Installing Terraform

Terraform is the infrastructure-as-code tool used to interact with cloud resources. In our case, we will be using it to provision a Kubernetes Cluster on Digital Ocean. Installation instructions can be found here: [https://www.terraform.io/downloads.html](https://www.terraform.io/downloads.html)

### Installing doctl

`doctl` is Digital Ocean's CLI tool to interact with Digital Ocean resources. This is required to authenticate their resources. This will also help us in setting up our   `kubeconfig` file.

### Installing kfctl

`kfctl` is a tool used to build and package Kubeflow and its [components](https://www.kubeflow.org/docs/components/). Download the latest release of `kfctl` from [here](https://github.com/kubeflow/kfctl/releases) and unpack it.

Creating A Kubernetes Cluster on Digital Ocean using Terraform
--------------------------------------------------------------

Creating a cluster using Terraform is straightforward. Make sure you have the latest version of Terraform installed.

1.  Create a file called `main.tf` for provider configuration, and add the following piece of code:

```bash
    terraform {
      required_providers {
        digitalocean = {
          source  = "digitalocean/digitalocean"
          version = "~> 2.0"
        }
      }
    }
    
    # Set the variable value in *.tfvars file
    # or using -var="do_token=..." CLI option
    variable "do_token" {}
    
    # Configure the DigitalOcean Provider
    provider "digitalocean" {
      token = var.do_token
    }
```

2\. Create a file called `cluster.tf` for cluster resource configuration, and add the following piece of code:

```bash
    resource "digitalocean_kubernetes_cluster" "k8s-challenge" {
      name   = "do-k8s-challenge"
      region = "nyc1"
      # Grab the latest version slug from `doctl kubernetes options versions`
      version = "1.19.15-do.0"
    
      node_pool {
        name       = "default"
        size       = "s-4vcpu-8gb"
        auto_scale = true
        min_nodes  = 1
        max_nodes  = 4
    
        tags = [
          "challenge",
          "kubeflow"
        ]
    
      }
    }
```

#### Note:

*   It is important to use an instance with higher compute and higher RAM because Kubeflow is resource hungry, and processes will quickly run out of CPU cores and memory, resulting in failure of installation of Kubeflow.
*   It is also important that we need 2-3 nodes. Autoscale is enabled in case you want to test a large ML workflow after installation. It is optional.
*   We are using version `1.19.15-do.0` because that is the latest stable version of Kubernetes on which Kubeflow will run properly.

3\. Create another file called `outputs.tf` for cluster creation outputs and logs if required. This file is optional and used for developer understanding only. Add the following content to the file:

```bash
    output "kubernetes_id" {
      description = "ID of the cluster"
      value       = digitalocean_kubernetes_cluster.k8s-challenge.id
    }
    output "kubernetes_host" {
      description = "The hostname of the API server for the cluster"
      value       = digitalocean_kubernetes_cluster.k8s-challenge.endpoint
    }
    
    output "kubernetes_urn" {
      description = "The uniform resource name (URN) for the Kubernetes cluster."
      value       = digitalocean_kubernetes_cluster.k8s-challenge.urn
    }
    
    output "kubernetes_created" {
      description = "Created at timestamp for the cluster"
      value       = digitalocean_kubernetes_cluster.k8s-challenge.created_at
    }
```

### Terraform configuration and setup

1.  In the working directory of the above files, run `terraform init` to initialize the repository with terraform provider dependencies listed in `main.tf` and you will see terraform configuration files, lock file and the state file being created.
2.  Create a file called `secret.auto.tfvars`. We will use this to store Digital Ocean access keys. Terraform will automatically load the contents of this file as terraform variables. You can obtain an access token from Digital Ocean using [this](https://docs.digitalocean.com/reference/api/create-personal-access-token/) link. The file should look like this:

    do_token = "YOUR_DO_ACCESS_KEY"

### Fixing a few bottlenecks before proceeding

Before we proceed, there are a few problems we might run into while we are installing Kubeflow. Here is a list of things to do before proceeding:

1.  Digital Ocean limits the number of droplets and volumes provided per account, per user. You can head to Digital Ocean's [cloud support](https://do-support.force.com/apex/DOUserAccessTokenCallbackSSOVF#access_token=0052604a3e5e48e547d22dde443786529d66c12cb2d4c83349ceb1fdb23562f6&token_type=bearer&expires_in=2592000&state=CAAAAX2xobRFMDAwMDAwMDAwMDAwMDAwAAAA6kYkWwfVGMdQ1PHQjFttDJnjc5lgqd7pH5-NG8Pty8n5xJcPssSDXYykrNHU1BZtwJt9jYJJlalxnWNGQGZQ_yt3CbettdRW0BJnfSsHFqcWGdrxZohnlcdEnvyNnUzeeDqbQyBtIkVVKRCQH9W1X3MQ8oikdvBimWLmhF7MO_G0OULOx6jFwf-O39N4dVmuXrrX3bNzHEzYyankQPTP7f3HNSjUD01cvGMrH1jXvcH8DsGHCm9Y4ivAXFnVA7DoVN_maBdrOl39MYGASdDU0JHPO0BY1eJShsu4kmc8mChc) and request to increase your instances limit as well as volumes limit (ideally 3-4 each)
2.  You might also need to put in your credit card as a part of their verification process. If you're a student and part of [GitHub Student Developer Pack](https://education.github.com/pack), you can get around $50 in credits which is sufficient to install and test Kubeflow.

### Creating the cluster

1.  If you have followed everything so far, we are ready to create our cluster. In the same working directory as the terraform files, run `terraform plan -out plan-1.out`
2.  This will create a plan, which tells you the resources that you are about to create or modify that will be reflected in Digital Ocean. Carefully observe the resources that will be created.
3.  Finally run `terraform apply "plan-1.out"`. This command will create the cluster. It might take around 5-10 mins for the cluster to be created.
4.  You can login to Digital Ocean's console and check if the cluster has been created.

### Setting up .kubeconfig

1.  Note down the cluster ID that you will find after the cluster has been created.
2.  To connect and setup our `.kubeconfig`, we need to run `doctl kubernetes cluster kubeconfig save YOUR_CLUSTER_ID`

### Verifying cluster creation

1.  Run `kubectl get namespaces` to see if everything worked.

You should now see the list of system-created namespaces, by Digital Ocean

Installing Kubeflow
-------------------

It is important to understand that Kubeflow consists of multiple components. Each of these components have sub-components that work with each other. This article assumes that one is aware of Kubeflow components such as Istio, Dex and pipelines. There is a good chance that the installation might fail because of the nature of the platform. Be sure to follow the steps carefully.

### Installing Kubeflow on Digital Ocean Kubernetes

1.  Recall that we installed a tool called `kfctl` earlier. Create a folder called `kubeflow` and set path as `export PATH=$PATH:"<path-to-kfctl>` on your CLI so that we can use `kfctl` as is.
2.  Set another variable called `export KF_NAME=do-kubeflow`. This is important for `kfctl` to know where it must store its manifest files.
3.  Now, set the base directory variable `BASE_DIR=<path to a base directory>` and, Kubeflow installation directory as `export KF_DIR=${BASE_DIR}/${KF_NAME}`. Run `mkdir -p ${KF_DIR}` to create the directory.
4.  Download the manifest file provided by Kubeflow from here: [https://raw.githubusercontent.com/kubeflow/manifests/v1.2-branch/kfdef/kfctl\_istio\_dex.v1.2.0.yaml](https://raw.githubusercontent.com/kubeflow/manifests/v1.2-branch/kfdef/kfctl_istio_dex.v1.2.0.yaml) and place it in the `do-kubeflow` directory you just created.
5.  Navigate to `do-kubeflow` directory and run `kfctl build -f kfctl_istio_dex.v1.2.0.yaml`. You will now see a set of files and folders being created. These are the manifest files useful for installation. Do not edit or move these files until the installation is complete.
6.  Run `kfctl apply -V -f kfctl_istio_dex.v1.2.0.yaml` to start the installation process. The process should take around 5-10 mins.
7.  Run `kubectl get all -n kubeflow` to see the pods running and to check the installed components. You might see some failed pods initially but that's just Kubeflow waiting for all components to connect to each other properly. Wait for a couple of minutes and then you'll be able to see all the pods up and running.

### Accessing the dashboard

*   Execute `kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80`

### Attaching a Load Balancer

*   Run `kubectl patch svc istio-ingressgateway -n istio-system -p '{"spec":{"type":"LoadBalancer"}}`

[Previous issue](https://sudhanva-narayana.ghost.io/analyzing-data-for-a-food-tech-startup-swiggy/)

[Browse all issues](https://sudhanva-narayana.ghost.io/page/2)

[Next issue](https://sudhanva-narayana.ghost.io/install-tensorflow-on-apple-m1-pro-max/)