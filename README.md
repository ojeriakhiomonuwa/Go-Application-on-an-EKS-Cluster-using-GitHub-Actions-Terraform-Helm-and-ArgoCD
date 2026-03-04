# Deploy a Go Application on an EKS Cluster using GitHub Actions, Terraform,  and ArgoCD
![](images/Untitled%20Diagram%20(1).gif)
 To thrive in today’s fast-paced tech landscape, mastering streamlined software lifecycles through DevOps is essential. As a DevOps Engineer, building a high-impact portfolio that showcases these competencies is the best way to demonstrate your expertise.

 In this project, we explore how to construct a sophisticated DevOps ecosystem using industry-leading tools like AWS EKS, GitHub Actions, Terraform, and ArgoCD. By the end of this guide, you will have implemented a fully automated GitOps pipeline that deploys a Go-based application to a private EKS cluster. We will cover everything from initial code quality checks and Docker image orchestration to vulnerability scanning with Docker Scout and real-time Slack notifications—all while managing deployments through native Kubernetes manifests for maximum transparency and control.
## Prerequisites
Before we dive into the details, ensure you have the following prerequisites in place:

* **AWS Account:** Set up an AWS account for creating and managing EKS clusters.
* **GitHub Account:** A GitHub account to host your repositories and configure GitHub Actions.
* **Terraform:** Installed on your local machine for managing infrastructure as code.
 **Kubectl:** Installed and configured to interact with your Kubernetes cluster.
* **AWS CLI:** Installed and configured to manage AWS resources from the command line.
* **Slack Webhook URL:** (Optional) Integrating Slack notifications into your CI/CD pipeline. 
### So, I have created three repositories for this project.

* ### EKS-Terraform-GitHub-Actions- Contains terraform code for EKS cluster and other required services along with GitHub actions workflow to deploy the infrastructure throw GitHub actions 
https://github.com/ojeriakhiomonuwa/EKS-Terraform-GitHub-Actions.git

* ### go-portfolio-project- Contains source code along with GitHub actions workflow including code analysis, docker image creation, image scanning, image tag updation, etc.  
https://github.com/ojeriakhiomonuwa/go-portfolio-project.git

* ### go-app-devops- Contains helm chart where the manifests are present to deploy it on the EKS Cluster.
https://github.com/ojeriakhiomonuwa/go-app-devops.git

 Deploy EKS using GitHub Actions
 https://github.com/ojeriakhiomonuwa/EKS-Terraform-GitHub-Actions.git

  Click on Actions
 ![](images/1.%20Deploy%20EKS%20using%20GitHub%20Actions%20Repo.png)

  Click on Terraform
 ![](images/2%20.png)

 Click on Run workflow and run the plan to validate what are we going to deploy
 ![](images/3.%20run%20workflow.png)

  The Plan is successful and you can click on Terraform-Action to view the blueprint of action
![](images/4.%20plan%20is%20successful.png)

 Total of 37(repo update 38) AWS resources will be created
![](images/5.%20total%20number%20of%20resources%20to%20be%20created.png)

 Now, we will run the apply  
 Go to Terraform workflow and click on Run Workflow
![](images/6.%20terraform%20apply.png)

 The Apply is successful and you can click on Terraform-Action to view the created resources list.

### You can validate whether the cluster has been created or not by going to the AWS console
![](images/7.%20eks%20created.png)

Once you try to connect with your EKS cluster on your local it will add the context.

But while running kubectl commands like get nodes, pods, etc you will get an error

If you look at the snippet, you will observe your local is unable to connect to your server

The reason behind this is your EKS cluster is Private and to access the cluster your server needs to be in the same VPC as the EKS cluster.
![](images/8.kube%20ctl%20get%20nodes%20error.png)

I have created one more instance and attached the same VPC that is used in the EKS Cluster.

You can refer to the below snippet
![](images/9.%20instance%20network%20settings.png)

Login to your server and do the following things mentioned below-

Install aws cli, kubectl

Configure aws cli by using the aws configure command and run the `kubectl get nodes` command
![](images/10.%20install%20aws%20cli.png)
![](images/11.%20kubectl%20get%20nodes.png)

Install helm which is one of our prerequisites  
`sudo snap install helm — classic`
![](images/12.%20install%20helm.png)

Install the Nginx ingress controller by using the below command  
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml`
![](images/13.%20install%20nginx%20ingress%20controller.png)

Validate whether your ingress controller pods are running or not. It should be running  
`kubectl get all -n ingress-nginx`
![](images/14.%20validate%20ingress%20controller.png)

Now, we need to install argoCD as per our project-required tool  
`kubectl create namespace argocd`  
`kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.7/manifests/install.yaml`
![](images/15.%20intsall%20argocd.png)

Validate whether argocd pods are running or not by using the below command    
`kubectl get pods -n argocd`
![](images/16.%20validate%20argocd%20pods.png)

Now, we need to expose our argocd service to the Loadbalancer type as we need to access it outside of our cluster  
`kubectl patch svc argocd-server -n argocd -p ‘{“spec”: {“type”: “LoadBalancer”}}’`
![](images/17.%20expose%20argocd%20to%20load%20balancer.png)

Validate whether the load balancer has been created or not by going to the AWS console
![](images/18.%20confirm%20loadbalancers%20.png)

Copy the DNS name of your argocd and hit on your favorite browser
![](images/19.%20dns%20name%20copied.png)

Click on Advanced as SSL is not configured on argoCD
![](images/20.%20dns%20error.png)
Here is your ArgoCD

Now, the username is admin but we don’t know the password.

To get the password, we need to run a few commands in the next steps
![](images/21.%20argocd%20active.png)

Now, we require a password to log in to the argoCD console.

The below command will list the secrets of argocd  
`kubectl get secrets -n argocd`
![](images/22.%20argocd%20secrets.png)

Run the below command and copy the password  
`kubectl edit secret argocd-initial-admin-secret -n argocd`
![](images/23.%20argocd%20password.png)

Run the below command to decode the password  
`echo <your-argocd-password> | base64 — decode`
![](images/24.%20decode%20argocd%20passwprd.png)

Now, log in to your argoCD console
![](images/25.%20login%20argocd.png)

We have set everything up to deploy our application
But there is one more thing that is connected to the repository in argoCD

### Why are we doing that?

So, our helm chart or manifest file is always stored in a private repository. To clone that repo, we need to set up our repository in argoCD.

Click on the settings icon on the left and then, click on Repositories
![](images/26.%20argocd%20settings.png)

For now, we are going to connect the repo using HTTPS which means we need a Personal Access token(PAT)

So, kindly generate the Personal Access token(PAT) of your GitHub account
![](images/27.%20argocd%20repo.png)

Provide the information accordingly
![](images/27.%20argo%20repo%20setting.png)

Once you see the connection status it means the repository has been connected successfully
![](images/28.%20connect%20the%20go-app%20git%20repo%20on%20argocd.png)

Create an application on argocd
![](images/29.%20create%20argo%20app.png)

Let’s create a workflow for our source code
![](images/29.%20workflow%20form%20source%20code.png)

Navigate to Repo’s Settings -> Security(on the left side) -> Secrets and variables and click on Actions

Create required secrets in which docker PAT, docker username, GitHub PAT, and slack webhook URL  

***Note:*** I am not configuring Slack notification, if you want to do it then, refer to this video https://www.youtube.com/watch?v=f8Tr3unIdNw and add a Slack webhook URL as well. But if you don’t want to integrate Slack then, you don’t need to add the secret. Also, skip the last job in the workflow.
![](images/29.%20update%20secrets.png)

This is our workflow code and you can check it out by clicking on the repo link  
https://github.com/ojeriakhiomonuwa/go-portfolio-project.git
![](images/30.%20our%20workflow%20code.png)

Now, we are set to run our workflow/pipeline.  
Workflow got successful
![](images/31.%20workflow%20was%20succesful.png)

The image has been pushed
![](images/32.%20image%20has%20been%20pushed.png)

Now, we will create an application on argocd to deploy our application on Kubernetes.

Provide the application name, repo URL, etc as shown in the below snippet
![](images/33.%20create%20go%20app.png)
![](images/34.%20createa%20go%20app.png)
![](images/35.%20create%20go%20app.png)

As soon as you create the application, it will be deployed.
![](images/36.%20go%20app%20created.png)

You can click on the application to view all the resources that have been created by argoCD
![](images/37.%20resources%20created%20.png)

You can also validate from your jump server by running the below command.  
`kubectl get all -n go-app`
![](images/38.%20validate%20from%20the%20instance.png)

Check the ingress address by using the below command  
`kubectl get ing -n go-app`
![](images/40.%20check%20ingress.png)

Go to your AWS console, navigate to the Load balancer, and copy the DNS name
![](images/39.%20copy%20dns%20name%20of%20load%20balancer.png)

Go to your domain provider and add A type record.

I am using AWS Route53 to add the record
![](images/41.%20create%20A%20record.png)

Once you add the record you can hit your domain on your favorite browser and see the magic
![](images/42.%20domain%20loads%20successfully.png)

### Conclusion
Congratulations! You’ve built a comprehensive DevOps portfolio that shows an end-to-end automated CI/CD pipeline for deploying a Go-based application on the EKS cluster. This case demonstrates your knowledge of Terraform, GitHub Actions, and ArgoCD. The presentation of this portfolio reveals not only your technical abilities but also the ability to automate complex workflows, which is a desired characteristic of any DevOps team. Keep adding new projects and tools to your portfolio and continue experimenting because DevOps is an evolving field.