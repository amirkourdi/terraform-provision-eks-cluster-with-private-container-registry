# terraform-provision-eks-cluster
Set up workspace
$ git clone https://github.com/AmirKourdi/terraform-provision-eks-cluster-with-private-container-registry

Customize deploymet 
1. vpc.tf provisions a VPC, subnets and availability zones using the AWS VPC Module. A new VPC is created for this example so it doesn't impact your existing cloud environment and resources.

2. security-groups.tf provisions the security groups used by the EKS cluster.

3. eks-cluster.tf provisions all the resources (AutoScaling Groups, etc...) required to set up an EKS cluster using the AWS EKS Module.

4. On line 14, the AutoScaling group configuration contains three nodes.

eks-cluster.tf
worker_groups = [
 {
   name                          = "worker-group-1"
   instance_type                 = "t2.small"
   additional_userdata           = "echo foo bar"
   additional_security_group_ids = [aws_security_group.worker_group_mgmt_one.id]
   asg_desired_capacity          = 2
 },
 {
   name                          = "worker-group-2"
   instance_type                 = "t2.medium"
   additional_userdata           = "echo foo bar"
   additional_security_group_ids = [aws_security_group.worker_group_mgmt_two.id]
   asg_desired_capacity          = 1
 },
]
4. outputs.tf defines the output configuration.

5. versions.tf sets the Terraform version to at least 0.14. It also sets versions for the providers used in this sample.

Initialize Terraform workspace
$ terraform init

Provision the EKS cluster
$ terraform apply

Configure kubectl
$ aws eks --region $(terraform output -raw region) update-kubeconfig --name $(terraform output -raw cluster_name)

Deploy Kubernetes Metrics Server
$ wget -O v0.3.6.tar.gz https://codeload.github.com/kubernetes-sigs/metrics-server/tar.gz/v0.3.6 && tar -xzf v0.3.6.tar.gz
$ kubectl apply -f metrics-server-0.3.6/deploy/1.8+/

 kubectl get deployment metrics-server -n kube-system
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
metrics-server   1/1     1            1           4s

Deploy Kubernetes Dashboard
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml

Acess dashboard
$ kubectl proxy

You should be able to access the Kubernetes dashboard here 
(http://127.0.0.1:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/).



# Docker Register
Creating basic auth credentials
First, we'll create our basic auth credentials file. You will need the htpasswd utility for this. On Ubuntu-based systems, this utility is packaged in the apache2-utils package. We can install that with a standard apt install command.

sudo apt install -y apache2-utils


Once the tool is installed, we can invoke it, giving it the name of the output file, htpasswd, and a username, in this case we'll use registry as our username. It will prompts us for a password, which we'll make up. We will also need to confirm the password.

htpasswd -Bc htpasswd registry
This will create the file htpasswd on our local file system.

We need to store this file in Kubernetes so that our docker registry can access it. We will do that by adding it as a Kubernetes secret. To do that, we use kubectl to create a secret named docker-registry-htpasswd from the local htpasswd file.

kubectl create secret generic docker-registry-htpasswd --from-file ./htpasswd

We can confirm that that worked
$ kubectl describe secret docker-registry-htpasswd
Name:         docker-registry-htpasswd
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====

Deploying the registry
kubectl apply -f registry.yaml

Validation
kubectl get pods

OUTPUT
NAME                              READY   STATUS    RESTARTS   AGE
docker-registry-695587f47-lz59h   1/1     Running   0          24m



