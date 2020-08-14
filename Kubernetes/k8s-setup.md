
#### Setup Kubernetes (K8s) Cluster on AWS


1. Create Ubuntu EC2 instance
1. install AWSCLI
   ```sh 
    curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
    sudo apt install unzip python
    unzip awscli-bundle.zip
    #sudo apt-get install unzip - if you dont have unzip in your system
    sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
    ```
    
1. Install kubectl
   ```sh
   curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
   ```
1. Create an IAM user/role  with Route53, EC2, IAM and S3 full access
1. Attach IAM role to ubuntu server

    #### Note: If you create IAM user with programmatic access then provide Access keys. If you are to root user just press enter for the next 3 commands (Access key, secret) region e.g. us-west-2
   ```sh 
     aws configure
    ```
1. Install kops on ubuntu instance:
   ```sh
    curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    chmod +x kops-linux-amd64
    sudo mv kops-linux-amd64 /usr/local/bin/kops
    ```
1. Create a Route53 private hosted zone (you can create Public hosted zone if you have a domain)
1. create an S3 bucket 
   ```sh
    aws s3 mb s3://dev.k8s.nerdstats.us
   ```
1. Expose environment variable:
   ```sh 
    export KOPS_STATE_STORE=s3://dev.k8s.nerdstats.us
   ```
1. Create sshkeys before creating cluster
   ```sh
    ssh-keygen
   ```
1. Create kubernetes cluster definitions on S3 bucket 
   ```sh 
    kops create cluster --cloud=aws --zones=us-west-2c --name=dev.k8s.nerdstats.us --dns-zone=nerdstats.ca --dns private
    ```
1. Create kubernetes cluser
    ```sh 
      kops update cluster dev.nerdstats.us --yes
     ```
1. Validate your cluster 
     ```sh 
      kops validate cluster
    ```

1. To list nodes
   ```sh 
     kubectl get nodes 
   ```

#### Deploying Nginx container on Kubernetes 
1. 
   ```sh
      kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
      kubectl get deployments
      kubectl get pods
    
#### Updating Nginx container updates on Kubernetes 
2.          
   ```sh
      kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record
      kubectl rollout status deployment.v1.apps/nginx-deployment
      kubectl get pods
      kubectl describe deployments
      kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.161 --record=true
      kubectl rollout status deployment.v1.apps/nginx-deployment
      kubectl get rs
      kubectl get pods
      kubectl describe deployment
   ```
 #### Rolling back Nginx container updates on Kubernetes 
3.      
   ```sh
      kubectl rollout history deployment.v1.apps/nginx-deployment
      kubectl rollout history deployment.v1.apps/nginx-deployment --revision=2
      kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision=2
      kubectl get deployment nginx-deployment
      kubectl describe deployment nginx-deployment
    ``` 
 #### Scaling Deployment on Kubernetes (Vertical, Horizonal)
4.       
   ```sh 
      kubectl scale deployment.v1.apps/nginx-deployment --replicas=10
      kubectl autoscale deployment.v1.apps/nginx-deployment --min=10 --max=15 --cpu-percent=80
  
#### Expose a deployment as a service Kubernetes 
5. Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them:
   ```sh 
    kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
    kubectl get services -o wide
  
#### Delete cluster Kubernetes 
 6. 
    ```sh
     kops delete cluster dev.k8s.valaxy.in --yes
  
