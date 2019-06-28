#
EKS Cleanup
* [Procedure link](https://docs.aws.amazon.com/eks/latest/userguide/delete-cluster.html)
1. Delete Worker Node CloudFormation stack
1. Delete Cluster
1. Delete VPC CloudFormation stack

EKS Verify Cleanup
* Ensure following services don't have resources
* [ASG](https://us-east-2.console.aws.amazon.com/ec2/autoscaling/home?region=us-east-2#AutoScalingGroups:)
* [Launch Configurations](https://us-east-2.console.aws.amazon.com/ec2/autoscaling/home?region=us-east-2#LaunchConfigurations:)
* [CloudFormation Stacks](https://us-east-2.console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks)
* [EKS Clusters](https://us-east-2.console.aws.amazon.com/eks/home?region=us-east-2#/clusters)
* [Sec Grps](https://us-east-2.console.aws.amazon.com/ec2/v2/home?region=us-east-2#SecurityGroups:)
* [Worker Instances](https://us-east-2.console.aws.amazon.com/ec2/v2/home?region=us-east-2#Instances:)
* [IAM Roles](https://console.aws.amazon.com/iam/home?region=us-east-2#/roles) - search for eks

EKS Setup
* [Procedure link](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html)

1. **Clone this repo**
    * $ git clone 
1. **Create IAM Role**
    * [IAM Console - Roles - KT-AWS-EKS-Week26](https://console.aws.amazon.com/iam/home#/roles)
    * The role needs only to be created once. Can be reused to create more clusters.
1. **Create VPC**
    * Execute [this CloudFormation template](https://github.com/gollamudiorg/aws-kt/tree/master/eks/eks-cluster-vpc.yaml) via the [AWS CF Console](https://us-east-2.console.aws.amazon.com/cloudformation)
1. **Create the Control Plane** 
    * Using [AWS ECS Console](https://us-east-2.console.aws.amazon.com/eks/home)
    * Cluster name *{your prefix}*-eks-cluster
1. **Install kubectl**
    * Linux
        * Download from [here](https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/linux/amd64/kubectl) into the project's root directory
        * Make it executable *$ chmod +x ./kubectl*
        * Check successful installation *$ kubectl help*
    * Windows
        * Download from [here](https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/windows/amd64/kubectl.exe) into the project's root directory
        * Check successful installation *$ kubectl help*
    * MacOS
        * Download from [here](https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/darwin/amd64/kubectl) into the project's root directory
        * Make it executable *$ chmod +x ./kubectl*
        * Check successful installation *$ kubectl help*
1. **Install AWS IAM Authenticator**
    * Linux
        * Download from [here](https://amazon-eks.s3-us-west-2.amazonaws.com/1.13.7/2019-06-11/bin/linux/amd64/aws-iam-authenticator) into the project's root directory
        * Make it executable *$ chmod +x ./aws-iam-authenticator*
        * Check successful installation *$ aws-iam-authenticator help*
    * Windows
        * Download from [here](https://amazon-eks.s3-us-west-2.amazonaws.com/1.13.7/2019-06-11/bin/windows/amd64/aws-iam-authenticator.exe) into the project's root directory
        * Check successful installation *$ aws-iam-authenticator help*
    * MacOS
        * Download from [here](https://amazon-eks.s3-us-west-2.amazonaws.com/1.13.7/2019-06-11/bin/darwin/amd64/aws-iam-authenticator) into the project's root directory
        * Make it executable *$ chmod +x ./aws-iam-authenticator*
        * Check successful installation *$ aws-iam-authenticator help*
1. **Configure kubectl to access the EKS ckuster**
    * Open your clusters General Configuration from the using [AWS ECS Console](https://us-east-2.console.aws.amazon.com/eks/home) by clicking on the cluster
    * Edit the [kubeconfig](https://github.com/gollamudiorg/aws-kt/tree/master/eks/kubeconfig) file to replace...
        * ${CLUSTER.endpoint} with the **Cluster's API Server Endpoint**
        * ${CLUSTER.ca-cert} with the **Cluster's CA certificate**
        * ${CLUSTER.name} with the **Cluster's name**
1. **Check that we can access the EKS cluster**
    * Run  *$ kubectl --kubeconfig kubeconfig get svc* to check communication with the cluster
1. **Provision worker nodes**
    * Execute [this CloudFormation template](https://github.com/gollamudiorg/aws-kt/tree/master/eks/eks-cluster-nodegroup.yaml) via the [AWS CF Console](https://us-east-2.console.aws.amazon.com/cloudformation)
        * Stack Name = *{your name}-eks-cluster-nodegroup*
        * ClusterName = *{your prefix}-eks-cluster*
        * ClusterControlPlaneSecurityGroup = select from drop down your cluster's VPC security group
        * NodeGroupName = *{your prefix}-eks-cluster-nodegroup*  
        * NodeImageId = select from [here](https://docs.aws.amazon.com/eks/latest/userguide/eks-optimized-ami.html) the AMI corresponding to K8s v1.13.x for your region
        * KeyPair = select your AWS key pair from the dropdown, to create a key pair follow the instructions [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair)
        * VpcId = select your cluster's VPC from the dropdown
        * Subnets = select your cluster's VPC subnets from the dropdown
1. **Add the worker nodes to the cluster**
    * Open your workers node group CloudFormation stack using the [AWS CF Console](https://us-east-2.console.aws.amazon.com/cloudformation) and go to the *Outputs* tab
    * Edit the [aws_auth_cm.yaml](https://github.com/gollamudiorg/aws-kt/tree/master/eks/aws_auth_cm.yaml) file to replace...
        * ${NodeGroup.NodeInstanceRole} with the NodeInstanceRole value
        * Use kubectl to apply the config map
            * *$ kubectl --kubeconfig kubeconfig apply -f aws-auth-cm.yaml*
1. **Check worker nodes successfully joined the cluster**
    * Run *$kubectl --kubeconfig kubeconfig get nodes* 
        
