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
- Adjust the Jenkinsfile of the previous CI/CD pipeline to configure the connection to the EKS cluster.

## 🏗 Project Architecture



## ⚙️ Project Configuration
### Installing kubectl and aws-iam-authenticator on a Jenkins server
1. Connect to the Jenkins server, which is hosted on digitlOcean using SSH
   ```bash
   ssh root@198.199.70.18
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CD_pipeline/blob/main/Img/100%20SSH%20to%20Jenkins%20server%20%20hosted%20%20in%20DO.PNG" width=800 />
   
2. Verify that the Jenkins container is running:
   ```bash
   docker ps
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CD_pipeline/blob/main/Img/101%20checking%20docker%20ocntainer%20is%20running.png" width=800 />
   
3. Access the Jenkins container as the root user:
   ```bash
   docker exec -u 0 -it 6db6fdd7ed8f bash
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CD_pipeline/blob/main/Img/102%20accessing%20container.png" width=800 />
   
 4. Install kubectl inside the Jenkins container:
    ```
    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl; chmod +x ./kubectl; mv ./kubectl /usr/local/bin/kubectl
    ```
    <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CD_pipeline/blob/main/Img/103%20installing%20kubectl%20in%20docker%20continer%20in%20jenkins%20server.png" width=800 />
    
5. Confirm that kubectl is installed:
   ```
   kubectl version
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CD_pipeline/blob/main/Img/104%20kubectl%20available%20in%20jenks.png" width=800 />
   
7. Install aws-iam-authenticator, set the appropriate permissions, and move it to the binary path: usr/local/bin
   ```
     curl -Lo aws-iam-authenticator https://github.com/kubernetes-sigs/aws-iam-authenticator/releases/download/v0.6.11/aws-iam-authenticator_0.6.11_linux_amd64
     chmod +x ./aws-iam-authenticator
     mv ./aws-iam-authenticator /usr/local/bin
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CD_pipeline/blob/main/Img/105%20installing%20aws%20authenticator%20in%20dockr%20jenkins%20server.png" width=800 />
   
### Creating the kubeconfig File Inside the Jenkins Container
The kubeconfig file is essential for authenticating and interacting with the EKS cluster. It contains the necessary credentials and configuration details to access the cluster using the associated AWS account. When the EKS cluster is created locally, the .kube/config file is automatically generated. To enable access from within the Jenkins Docker container, this file must also be present inside the container. Since the Jenkins container is lightweight and lacks a text editor, you should generate the kubeconfig file on the host machine and then copy it into the container if using the first approach.

1. Access the Jenkins server host.
2. Create the config file:
   ```bash
   vim config
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CD_pipeline/blob/main/Img/106%20a%20cretae%20the%20config%20file.PNG" width=800 />
   
3. Add the following configuration, replacing the placeholders with your cluster information: update the cluster name, certificate-authority-data, and server.
   * Cluster name: Your EKS cluster name.
   * server: API server endpoint (from the EKS cluster details).
   * certificate-authority-data: Found in your local .kube/config file.

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
     
4. Save the file.
5. Enter the Jenkins container as Jenkins User
  ```bash
    docker exec -u 0 -it <container_id> bash
    docker exec -it 6db6fdd7ed8f bash
  ```
<img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CD_pipeline/blob/main/Img/108%20enter%20container%20as%20jenkins%20user.png" width=800 />

7. Create the .kube directory inside the Jenkins home directory:
   ```bash
     mkdir /var/jenkins_home/.kube
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CD_pipeline/blob/main/Img/109%20creating%20kube%20directory%20inside%20jenkins.png" width=800 />

   
8. Exit the container
   
10. Copy the config file from the Jenkins server into the Jenkins container:
   ```bash
   docker cp config 6db6fdd7ed8f:/var/jenkins_home/.kube/
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CD_pipeline/blob/main/Img/110%20copying%20comfig%20file%20from%20jenkins%20droplet%20to%20the%20container.png" width=800 />
   

11. Enter the Jenkins container as Jenkins User
    ```bash
      docker exec -it 6db6fdd7ed8f bash
    ```
    <img src="" width=800 />

11. Verify the config file is available in the .kube directory
  ```bash
    ls -la
  ```
<img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CD_pipeline/blob/main/Img/111%20checking%20config%20file%20inside%20the%20conntainer.png" width=800 />

### Adding AWS credentials to Jenkins for AWS authentication
 
1.  In the Jenkins Dashboard, go to Manage Jenkins > Credentials.
2.  Create new Global Credentials:
   * Add the jenkins_aws_access_key_id as a secret type. The secret is the actual key ID.
   * Add the jenkins-aws_secret_access_key as Secret text type. The secret is the Access Key.

  <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CD_pipeline/blob/main/Img/113%20create%20the%20secret%20access%20key%20and%20access%20key%20id%20for%20aws%20in%20jenkins.PNG" width=800 />

### Updating Jenkinsfile
1. In your Java-Maven-App repository, create a new feature branch named deploy-on-k8s.
2. Update the Jenkinsfile and update the deployment stage, create the nginx deployment
   ```bash
     sh 'kubectl create deployment.yaml nginx-deployment --image=nginx'
   ```                                                             
3. Define the environment variables that will be used in the connection.
   ```bash
     stage('deploy'){
       environment{
         AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
         AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws_secret_access_key')
     }
     steps{
       script {
         echo 'deploying docker image...'
         sh 'kubectl get nodes'
         sh 'kubectl create deployment nginx-deployment --image=nginx'
       }
     }
   }
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CD_pipeline/blob/main/Img/114%20setting%20up%20env%20for%20amazon%20authentication%20and%20deployment%20using%20kubectl.png" width=800 />

<details><summary><strong> Using AWS Pluging </strong></summary>
 If you encounter certificate verification issues such as: "ls: failed to verify certificate: x509: certificate signed by unknown authority". You can use the AWS Credentials Plugin to manage AWS authentication more securely and reliably.
Prerequisites:
Ensure the Jenkins agent or container has:
  <li> AWS CLI installed.</li>
  <li> kubectl installed.</li>
  <li> Access to the required AWS IAM role.</li>
  <li> EKS cluster name and region are defined in the environment.</li>


  1. Install the AWS Credentials plugin:
     
  2. Go to Manage Jenkins > Plugins > Available Plugins.
     
  3. Search for AWS Credentials.
     <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CD_pipeline/blob/main/Img/115%20using%20amazon%20credentialns%20intead%20after%20facing%20issue%20with%20pipeline.png" width=800 />
     
  5. Add a new Global Credentials:
     * Kind: AWS Credentials
     * Enter the Key ID, description, and the Access Key.

      <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CD_pipeline/blob/main/Img/116%20aws%20crdentials%20step.png" width=800 />
       
  6. Update the Jenkinsfile to use the AWS credentials plugin
      ```bash

              stage('deploy') { 
                    steps {
        
                        withCredentials([[
                            $class: 'AmazonWebServicesCredentialsBinding',
                            credentialsId: 'AWS_jenkins_key'
        
                        ]]) {
        
                            script {
                                echo 'Generating kubeconfig...'
                                
                                sh '''
                                    aws eks update-kubeconfig \
                                      --region $AWS_REGION \
                                      --name $CLUSTER_NAME \
                                      --kubeconfig $KUBECONFIG
                                '''
        
                                echo 'deploying docker image...'
                                sh 'kubectl get nodes'
                                sh 'kubectl create deployment nginx-deployment --image=nginx'
                            }
                        }
                    }
                }
      ```
7. Export the environment variables: In this case, we generate the kubeconfig file in the pipeline. This requires Jenkins agents to have:

  * AWS CLI installed
  * AWS credentials configured (via withCredentials)
  * EKS Region and cluster name available

  ```bash
     environment {
        KUBECONFIG = "${env.WORKSPACE}/kubeconfig"
        AWS_REGION = 'us-east-2'
        CLUSTER_NAME = 'demo-cluster'

    }

  ```
<img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CD_pipeline/blob/main/Img/fixed%20using%20aws%20credentials%20steps.PNG" width=500 />
  
</details>

   


   




