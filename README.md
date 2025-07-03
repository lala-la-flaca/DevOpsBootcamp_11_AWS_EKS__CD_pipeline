# AWS EKS
## 📦 Demo 3
This project is part of **Module 11: Kubernetes on AWS (EKS)** in the **TWN DevOps Bootcamp**. The goal of this demo is to implement CD by automatically deploying a containerized application to an **Amazon EKS cluster** using a **Jenkins pipeline**.

## 📌 Objective


## 🚀 Technologies Used
- **Amazon EKS**: Amazon Elastic Kubernetes Service.
- **Amazon EC2**: Compute Instances used for worker nodes.
- **I AM**: AWS identity service to manage access and secure permissions.
- **kubectl**: CLI to interact with Kubernetes.
- **AWS CLI**: Interface for managing AWS services
- **CloudFormation**: To manage the VPC template.
- **eksctl**: CLI tool for creating and managing EKS clusters
- **DigitalOcean**: hosting Jenkins server
  
## 📋 Prerequisites
- Ensure you have an AWS Account.
- Ensure AWS CLI is installed and configured.
- Kubectl is installed and configured to connect to the Kubernetes cluster.
- Jenkins server is running
- We are going to use the java-mave-app repository from previous demo.
  
## 🎯 Features
- Configure kubectl and aws-iam-authenticator on a Jenkins server.
- Create a kubeconfig file to connect to the EKS cluster and add it to the Jenkins Server.
- Add AWS credentials on Jenkins for AWS authentication
- Adjust JenkinsFile of the previous CI/CD pipeline to configure connection to EKS cluster.

## 🏗 Project Architecture



## ⚙️ Project Configuration
### Installing kubectl and aws-iam-authenticator on a Jenkins server
1. Connect to the Jenkins server which is hosted on digitlOcean using SSH
   ```bash
   ssh root@198.199.70.18
   ```
2. Verify the running containers
   ```bash
   docker ps
   ```
3. Access the Jenkins Container as a root user
   ```bash
   docker exec -u 0 -it 6db6fdd7ed8f bash
   ```
 4. Install kubectl inside the container
    ```
    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl; chmod +x ./kubectl; mv ./kubectl /usr/local/bin/kubectl
    ```
5. Verify that kubectl is installed
   ```
   kubectl version
   ```
7. Install aws-iam-authenticator, grant execute permission and move the script to the usr/local/bin
   ```
     curl -Lo aws-iam-authenticator https://github.com/kubernetes-sigs/aws-iam-authenticator/releases/download/v0.6.11/aws-iam-authenticator_0.6.11_linux_amd64
     chmod +x ./aws-iam-authenticator
     mv ./aws-iam-authenticator /usr/local/bin
   ```
### Creating the kubeconfig file in the Jenkins container.
The kubeconfig file contains all the necessary information for authentication using the AWS account and the EKS cluster. On the local machine, when we created the EKS cluster, the .kube/config was automatically created. Therefore, we must create the same file inside Jenkins Docker container to be able to access AWS account and EKS cluster. The jenkins container is a lightweight and does not have an editor. Therefore, we must create the file outside and copy it to the container.

1. Access to the Jenkins server
2. Create the config file
   ```bash
   vim config
   ```
3. Copy the content from the AWS documentation and update the cluster name, certificate-authority-data, and server.
   Cluster name: EKS cluster created
   server: API server endpoint (Available on the EKS cluster information)
   certificate-authority-data: valid token ( this is available on the local machine .kube/config file)
     ```bash
             apiVersion: v1
             kind: Config
             clusters:
             - cluster:
                  certificate-authority-data: <certificate-data>
                  server: <endpoint-url>
               name: kubernetes
             contexts:
             - context:
                  cluster: kubernetes
                  user: aws
                name: aws
             current-context: aws
             users:
             - name: aws
               user:
                  exec:
                    apiVersion: client.authentication.k8s.io/v1beta1
                    command: /usr/local/bin/aws-iam-authenticator
                    args:
                      - "token"
                      - "-i"
                      - <cluster-name>
   ```
     
4. Save changes
5. Enter the Jenkins container
6. Create the .kube directory at the default kubeconfig location (.kube/config) at the Jenkins home directory
   ```bash
     mkdir .kube
   ```
8. Copy the config file from the jenkins server to the jenkins container
   ```bash
   docker cp config 6db6fdd7ed8f:/var/jenkins_home/.kube/
   ```
9. Enter the Jenkins container as Jenkins User
    ```bash
      docker exec -it 6db6fdd7ed8f bash
   ```

10. Verify the config file is available in the .kube directory


### Adding AWS credentials on Jenkins for AWS authentication
 
1.  Go to the Jenkins server, in the dashboard under Manage Jenkins go to Credentials
2.  Create a new global credentials
3.  Add the jenkins_aws_access_key_id as a secret type, and the secret is the actual key ID.
4.  Add the jenkins-aws_secret_access_key as Secret text type, and the secret is the key

### Adjusting Jenkinsfile
1. Go to the java-maven-app project and create a new feature branch, named deploy-on-k8s
2. Update the Jenkinsfile on the deployment section, create the nginx deployment
   ```bash
     sh 'kubectl create deployment.yaml nginx-deployment --image=nginx'
   ```                                                             
3. Define the environment variables that will be used in the connection.
   ```bash
   environment{
   

   }
   ```
   




