# How to Create an EKS Cluster with Terraform

## What is trying to be done?
This will show you how to 1) deploy an EKS cluster using Terraform and 2) Configure `kubectl` using `Terraform output` to deploy a Kubernetes dashboard on the cluster.

Terraform is a great way of deploying your infrastructure accross various providers. Here you will see how easily it can be done on AWS

## Assumptions
- You have Terraform installed on your system.
- You have AWS CLI installed on your system with access keys set up for an appropriate user profile.

### How to setup the project
Steps:
- First clone the repository 
- `cd` into the directory 
- run `terraform init` 

And you're on your way!

## What will be created?
This terraform example will create a new VPC in US-East-1 region with:
- 3 public and 3 private subnets. 
- An EKS Cluster
- ASG of Worker EC2 instances with in private subnets connected to NAT Gateway in public subnet
- necessary Security Groups to allow SSH port 22
- internet gateway
- IAM Roles 


![Untitled Diagram](https://user-images.githubusercontent.com/62077185/125118484-c399a800-e0bd-11eb-8f31-1aed00bbd542.png)

**Create an ASG of Bastion Hosts manually to SSH - OR - install SSM Sessions Manager on EC2 instances without needing to SSH into instances (Instance IAM roles would need to be modified and the Security Groups can be changed as access to port 22 wouldn't be required).**

### Next step?
All that needs to be done now is run `terraform apply`
You will see the full list of everything that will be created with the EKS module.

Select `yes` in order to proceed with creating the project.

### Wait for the cluster....
Creating all of these resources should take 5-10 minutes so be patient


## How to Use Your Project
OK great the resources have been created! Now time to set things up with `kubectl`.
Terraform output created outputs for us to simplify the next step...
You will need to run the following command in order to get the necessary credentials to properly set up `kubectl`
Run: `aws eks --region $(terraform output -raw region) update-kubeconfig --name $(terraform output -raw cluster_name)`

# Setting up the Kubernetes Dashboard
Now let's see if everything is working as it should by setting up the Kuberntes Dashboard. We will do this next step using `kubectl` instead of using Terraform. 

## Downloand and deploy the Dashboard
Run the following: `wget -O v0.3.6.tar.gz https://codeload.github.com/kubernetes-sigs/metrics-server/tar.gz/v0.3.6 && tar -xzf v0.3.6.tar.gz`
And now using `kubectl`:
Deploy the metrics server: `kubectl apply -f metrics-server-0.3.6/deploy/1.8+/`

Now check on the deployment: `kubectl get deploy metrics-server -n kube-system` to make sure it deployed properly.

If everything is running as it should proceed to the next step.

## Schedule Necessary resources for the Dashboard
Run: `kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml`

Alright. Easy enough. Now a proxy server needs to be started and kept running and this can be done using the next command:
`kubectl proxy`

The link below should now let you access the dashboard: 
`http://127.0.0.1:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/`


## Finished?
Well no. You may be able to open the link but the Dashboard needs authentication. 

### Authenticating Kubernetes Dashboard
A cluster role binding needs to be created to provide an authorization token. This gives whoever the cluster-admin is the necessary permission needed to access the dashboard. Authenticating using kubeconfig is not an option here but please have a read over of the official kubernetes documentation. `https://kubernetes.io/docs/home/`

Make sure not to close the proxy process by opening a new terminal in order to create the cluster role binding.

`kubectl apply -f https://raw.githubusercontent.com/hashicorp/learn-terraform-provision-eks-cluster/master/kubernetes-dashboard-admin.rbac.yaml`

Now that is done, let's create the token needed...

`kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep service-controller-token | awk '{print $1}')`

Copy the newly created token and use it to sign into the dashboard by clicking on `Token` on the Kubernetes Dashboard. 
This should sign you into the cluster. Well done! Have a look around!

## Finished!
See that was easy! An EKS cluster and Kubernetes Dashboard were created in no time at all. Terraform really simplifies the process of setting up infrastructure. It's just as easy to destroy as well. Enjoy!
`terraform destroy`

