Here’s a **brief step-by-step guide** to installing a cluster on **user-provisioned infrastructure in AWS** **without** using **CloudFormation templates**:

[Documentation Author: Ankit Jain](https://www.linkedin.com/in/ankitkkjain/)

### **1. Prerequisites**
- Ensure you have an **AWS account** and necessary **IAM permissions**.
- Install **AWS CLI**:
  ```bash
  sudo apt install awscli  # Debian-based systems
  sudo dnf install awscli  # Fedora-based systems
  ```
- Install **OpenShift Installer**:
  ```bash
  curl -O https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/latest/openshift-install-linux.tar.gz
  tar -xvf openshift-install-linux.tar.gz
  ```

### **2. Manually Create AWS Infrastructure**
- Create a **VPC**:
  ```bash
  aws ec2 create-vpc --cidr-block 10.0.0.0/16
  ```
- Set up **subnets**:
  ```bash
  aws ec2 create-subnet --vpc-id <vpc-id> --cidr-block 10.0.1.0/24
  ```
- Configure **security groups**:
  ```bash
  aws ec2 create-security-group --group-name my-security-group --description "My security group" --vpc-id <vpc-id>
  ```
- Create **IAM roles**:
  ```bash
  aws iam create-role --role-name OpenShiftNodeRole --assume-role-policy-document file://trust-policy.json
  ```

### **3. Deploy Cluster Nodes**
- Create the **bootstrap node**:
  ```bash
  aws ec2 run-instances --image-id <ami-id> --count 1 --instance-type m5.large --key-name my-key --security-group-ids <sg-id> --subnet-id <subnet-id>
  ```
- Deploy **control plane nodes**:
  ```bash
  aws ec2 run-instances --image-id <ami-id> --count 3 --instance-type m5.large --key-name my-key --security-group-ids <sg-id> --subnet-id <subnet-id>
  ```
- Deploy **worker nodes**:
  ```bash
  aws ec2 run-instances --image-id <ami-id> --count 3 --instance-type m5.large --key-name my-key --security-group-ids <sg-id> --subnet-id <subnet-id>
  ```

### **4. Install OpenShift**
- Generate the **install-config.yaml** file:
  ```bash
  openshift-install create install-config
  ```
- Start the installation:
  ```bash
  openshift-install create cluster
  ```
- Monitor the installation logs:
  ```bash
  openshift-install wait-for bootstrap-complete --log-level=debug
  ```
- Approve **certificate signing requests (CSRs)**:
  ```bash
  oc get csr
  oc adm certificate approve <csr-name>
  ```
- Verify the cluster is running:
  ```bash
  oc get nodes
  ```

For detailed instructions, check out [Red Hat’s official documentation](https://docs.okd.io/4.8/installing/installing_aws/installing-aws-government-region.html) or [AWS installation options](https://docs.aws.amazon.com/prescriptive-guidance/latest/red-hat-openshift-on-aws-implementation/installation-options.html).
