# Deploy-App-using-EKS
## Deploy application in AWS EKS using CloudFormation

Though containers have been around since the early days of Linux, Docker introduced the modern version of this technology. Kubernetes is open source software that allows you to deploy and manage containerized applications at scale. It also provides portability and extensibility, enabling you to scale containers seamlessly.
### Steps invloved on deploying Flask application in AWS EKS using CloudFormation
#### Step 1 Create an AWS IAM role and VPC using Cloud Formation.
    1. Navigate to AWS IAM Console & in “Roles” section, click the “Create role” button
    2. Select “AWS service” as the type of entity and “EKS” as the service
    3. Enter a name for the service role and click “Create role” to create the role 
Amazon EKS also requires a Virtual Private Cloud (VPC) to deploy the cluster. To create this VPC:

    4. Navigate to the AWS CloudFormation console and click on “Create Stack”
    5. On the “Select Template” page, select the option to “Specify an Amazon S3 template URL” and enter the URL
    6. After specifying all the details, review and confirm them. Then click “Create” to proceed
#### Step 2 Create an Amazon EKS cluster
    1. Navigate to the Amazon EKS console and click on “Create cluster” button
    2. Enter details into the EKS cluster creation form such as cluster name, role ARN, VPC, subnets and security groups
    3. Click “Create” to create the Amazon EKS cluster
#### Step 3 Configure kubectl for Amazon EKS cluster 
Kubernetes uses a command line called Kubectl for communicating with kubernetes cluster. Amazon EKS clusters also require AWS IAM authentication for kubernetes to allow IAM authenticator for your kubenetes cluster.
###### Commands to install kubectl
     curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/arm64/kubectl
   now create Kubeconfig file for cluster with AWS CLI update-kubeconfig
   now test configuration
#### Step 4 Launch & Configure amazon EKS worker nodes
    1. Navigate to the AWS CloudFormation console and click on “Create stack” option
    2. On the “Select Template” page, select the option to “Specify an Amazon S3 template URL” and enter the URL
    3. Once stack creation is complete, select the stack name in the list of available stacks and select the “Outputs” section in the lower left pane.
    4. Now create a file aws.yaml, open it and add the following:
            apiVersion: v1
            kind: ConfigMap
            metadata:
               name: aws
               namespace: kube-system
            data:
               mapRoles:
                - rolearn: <AWS-ARN>
                  username: system:node:{{EC2PrivateDNSName}}
                  groups:
                    - system:bootstrappers
                    - system:nodes
Watch the status of node till then to reach the ready state.
#### Step 5 Deploy Simple hello world application
    Create helloworld.yaml file and add the following:
        apiVersion: extensions/v1beta1
        kind: Deployment
        metadata:
           name: hello-world
        spec:
           replicas: 1 
           template:
             metadata:
                labels:
                    app: hello-world
             spec:
                containers:
                    - name: hello-world-pod
                      image: krishnachaitanya/helloworld-spring-boot:latest
                      ports: 
                         - containerPort: 80
                         - containerPort: 443
         ---
          apiVersion: v1
          kind: Service
          metadata:
              name: hello-world-service
          spec:
            selector:
               app: hello-world 
            ports:
            - name: http
              protocol: TCP
              port: 80
              targetPort: 8080
            type: LoadBalancer
  
Save and Exit.

###### Now create nginx-svc.yaml file.
        Open it and add the following.
          apiVersion: v1
            kind: Service
            metadata:
               name: nginx
               labels:
                 run: nginx
            specs:
               ports:
                  - port : 80
                    protocol : TCP
               selector:
                  run: nginx
               type: LoadBalancer
               
###### Now check if the hello world application and nginx service is created or not by using following command:
            Kubectl create -f helloworld.yaml
            Kubectl create -f nginx-svc.yaml
Both should be created.
###### To check the nginx service you can give command
            kubectl describe svc ngnix
 by giving this command you will get load balancer ingress, copy that and paste the URL on browser.
 ###### You will see output on the webpage
            "hello world"



