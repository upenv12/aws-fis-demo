# AWS Fault Injection Simulator (FIS) examples

This repo contains a CloudFormation template with examples for AWS FIS. Experiment Templates for the following FIS experiments are included:
- Stopping a single instance by id
- Aborting an experiment when a CloudWatch Alarm is triggered
- Stopping all instances in a single Availability Zone
- Injecting CPU stress via ssm:send-command
- Terminate an EKS Worker Node


## Deployment

- This deployment will create 6 EC2 instances, spread across 2 AZs
- An EKS cluster with managed node groups

### Deploy via Cloudformation:

```bash
aws cloudformation create-stack --template-body file://aws_fis_demo.yaml --stack-name aws-fis-demo --capabilities CAPABILITY_NAMED_IAM
```
#### Deploy EKS Cluster
EKS cluster creation is not part of the above CloudFormation template. You will have to create the EKS cluster with managed node groups using the following command: 

If `eksctl` is not already installed, you can install it as follows:
```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/0.44.0/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp

sudo mv -v /tmp/eksctl /usr/local/bin
```

Create the EKS Cluster
```bash
eksctl create cluster -f eks_fis_cluster.yml
```

## Run Experiments

### Stop Single Instance By InstanceId
 - Start the FIS experiment `StopAndStartInstance` via console.
 - This will stop the EC2 instance named `FisExampleStack/instance-0` and then restart it after 1 minute.

### Abort experiment when alarm is raised
- Start the FIS experiment `AbortFiSExperimentByAlarm` from the console.
- This will stop the EC2 instance named `FisExampleStack/instance-0`
- Raise the CloudWatch alarm via CLI: 
```bash
aws cloudwatch set-alarm-state --alarm-name "NetworkInAbnormal" --state-value "ALARM" --state-reason "FIS Testing"
```
- The experiment will stop as soon as the alarm is triggered

### Stop All Instances in a single AZ
- Start the FIS experiment `StopInstancesInAz` from the console.
- All the EC2 instances in the given AZ will be stopped.
- After the FIS experiement is completed, you can restart the instances.

### Inject CPU stress via SSM
- Start the FIS experiment `StressCPUThroughSSM` from the console.
- This will induce stress into the EC2 instance named `FisExampleStack/instance-0`
- Connect to the targeted instance via SSM
- [Optional] Install `htop` utility to view the CPU usage details. This is similar to `top`, however gives a graphic view of CPU and Memory utilization. 
```bash
sudo yum install -y htop
```
- Note the high cpu load (e.g. either use `top` or `htop`)

### EKS Terminate Worker Node
- FIS currently supports fault injection into managed node groups. 
- Ensure that EKS cluster is created as per `eks-fis-cluster-yaml`, and the name of EKS cluster is `eks-demo`.
- Note: When creating the EKS cluster with eksctl, default tags are created. The name of the cluster will have the tag key as ``'eksctl.cluster.k8s.io/v1alpha1/cluster-name'`.
- Start the FIS experiment `Terminate EKS Workers` from the console. 
- Observe the termination of nodes. Since these are managed nodes, the ASG will bring up the number of nodes to desired state.


## Clean-Up

### Delete resources creaated with CloudFormation
Delete the CloudFormation stack via CLI:
```bash
aws cloudformation delete-stack --stack-name aws-fis-demo
```
### Delete EKS cluster if created
```bash
eksctl delete cluster -f eks_fis_cluster.yml
```

## ToDo:
- Document CLI commands for invoking FIS experiments. 