---
title: "Install JupyterHub on AWS Elastic Kubernetes Service (EKS)"
datePublished: Tue Mar 02 2021 04:30:00 GMT+0000 (Coordinated Universal Time)
cuid: cl7am8zgb02t8aznvbdb67gr0
slug: install-jupyterhub-on-aws-elastic-kubernetes-service-eks
canonical: https://sudhanva-narayana.ghost.io/install-jupyterhub-on-aws-elastic-kubernetes-service-eks/
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1661526347353/lFk-0ZQJs.jpeg
tags: aws, machine-learning, kubernetes, eks

---

Introduction
------------

*   Create a Docker image of all the libraries required for your workspace. This Docker image should be built on top of the Jupyter Notebook Docker images provided by Jupyter.org. Refer [https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html) for details of all the base images and the libraries that come installed with them
*   Push this custom Docker image to DockerHub
*   Inside the hub subdirectory, there is a config.yaml file. This file contains all the configurations for the deployed JupyterHub. Add your environment to the config.yaml file in the specified format. Refer [https://zero-to-jupyterhub.readthedocs.io/en/latest/customizing/user-environment.html#using-multiple-profiles-to-let-users-select-their-environment](https://zero-to-jupyterhub.readthedocs.io/en/latest/customizing/user-environment.html#using-multiple-profiles-to-let-users-select-their-environment) for examples on how to do the same.
*   Once you make your changes and commit, JupyterHub should restart and add the new environment in the JupyterHub launcher. Restart your user server from within JupyterHub to see the changes.

GPU enabled Docker image
------------------------

*   Use the image from [https://hub.docker.com/r/cschranz/gpu-jupyter](https://hub.docker.com/r/cschranz/gpu-jupyter) for GPUs to be recognised inside your environment (it's already in the config file, just choose that as the environment)
*   Other tensorflow images won't detect the GPUs due to issues with CUDA and cuDNN

Guidelines For a new installation
---------------------------------

### Initial Setup

[https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html)

*   Create a new group called admin
*   Assign Administrator Access policy
*   Create user, enable programmatic and AWS console access, and add to the admin group (if required)
*   MFA in the root account must be enabled. Assign Virtual MFA (Microsoft Authenticator)
*   Root account access key should be deleted
*   Create additional users and groups as necessary
*   Sign out of root account
*   Sign in as i-am-user
*   Cloud Formation Role needed to create any of the following components. This also needs IAM access

### Creating a VPC

[https://docs.aws.amazon.com/eks/latest/userguide/create-public-private-vpc.html](https://docs.aws.amazon.com/eks/latest/userguide/create-public-private-vpc.html)

*   Add the necessary policies to the I-AM role.
*   Create a VPC from cloud formation using the CF template
*   Default is Public-Private

Creating the Cluster
--------------------

[https://docs.aws.amazon.com/eks/latest/userguide/cluster-endpoint.html#modify-endpoint-access](https://docs.aws.amazon.com/eks/latest/userguide/cluster-endpoint.html#modify-endpoint-access)

*   Create cluster from the EKS console
*   Add security groups and subnets related to the previously created VPC
*   Security group has ControlPlaneSecurityGroup in the name available in the drop down
*   Configure cluster end point access. It is Public by default

### Attaching Nodes to the Cluster

Opening a ticket on AWS: AWS does not let you create GPU instances, so a ticket must be created for this

Nodes do not connect to cluster: This is because of a changed configuration from AWS. MapPublicIpOnLaunch should be set to true in the VPC settings.

*   [https://medium.com/@tarunprakash/5-things-you-need-know-to-add-worker-nodes-in-the-aws-eks-cluster-bfbcb9fa0c37](https://medium.com/@tarunprakash/5-things-you-need-know-to-add-worker-nodes-in-the-aws-eks-cluster-bfbcb9fa0c37)
*   [https://aws.amazon.com/blogs/containers/de-mystifying-cluster-networking-for-amazon-eks-worker-nodes/](https://aws.amazon.com/blogs/containers/de-mystifying-cluster-networking-for-amazon-eks-worker-nodes/)
*   [https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-subnet.html](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-subnet.html)
*   [https://gist.github.com/lizrice/5889f33511aab739d873cb622688317e](https://gist.github.com/lizrice/5889f33511aab739d873cb622688317e)
*   This is the command to check status of this setting  
    aws ec2 describe-subnets --filters "Name=vpc-id,Values=" | grep 'MapPublicIpOnLaunch|SubnetId|VpcId|State'

### JupyterHub Configuration:

*   [https://zero-to-jupyterhub.readthedocs.io/en/latest/index.html](https://zero-to-jupyterhub.readthedocs.io/en/latest/index.html)
*   [https://github.com/jupyterhub/zero-to-jupyterhub-k8s/issues/1637](https://github.com/jupyterhub/zero-to-jupyterhub-k8s/issues/1637) - Configuring Load Balancer on AWS to show JupyterHub UI, because   AWS Load Balancer points to the wrong port by default and does not run.
*   Change namespace to Jhub always, to run helm update commands for configuration changes in JupyterHub

[Previous issue](https://sudhanva-narayana.ghost.io/a-tool-that-automatically-selects-cheapest-cloud-platform-for-running-your-ml-code/)

[Browse all issues](https://sudhanva-narayana.ghost.io/page/2)

[Next issue](https://sudhanva-narayana.ghost.io/leaving-pixxel/)