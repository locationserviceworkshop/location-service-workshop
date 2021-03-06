# AWS provisioning & deployments

This directory contains two parts, *aws-setup* and *ansible*. 

- *aws-setup* contains playbooks to request a set of servers on the AWS EC2 cloud and install some basic tools.
- *ansible* contains the playbooks to configure the servers with docker, jenkins, etc.

### Creating servers in amazon

Next steps assume you are working on Linux and you have a working AWS account. 


**Setting up the environment in ansible**
 1. Switch to region Oregon (*us-west-2*). This region supports EKS (Elastic Container Service for Kubernetes).
 2. Create a new IAM (Identity and Access Management) user named *ansible-all* in Amazon AWS, 
    and attach the security policy `AmazonEC2FullAccess`, download the credentials. 
 3. Create another user ansible-read with `AmazonEC2ReadOnlyAccess`, download the credentials. 
 4. Go to Services > EC2 > Key Pairs > click create key pair. Name the key-pair *location-workshop-keypair*. Save the
    .pem file as well, you can access the servers via SSH using this key.     
 5. Install awscli, [see online docs](http://docs.aws.amazon.com/cli/latest/userguide/installing.html)
 
    ```bash
     sudo apt install python-pip
     pip install pip --upgrade
     sudo pip install awscli boto
     ```
    
 6. On the command line, execute: `aws configure` and configure the credentials of the account 
    with the credentials of ansible-all (which has AmazonEC2FullAccess rights), set region to *us-west-2* 
    and output format to json.
 7  Disable host key checking by `export ANSIBLE_HOST_KEY_CHECKING=False`
 8. Automatically create and start nodes in Amazon EC2 for each group using Ansible using the commands below. 
    Open provistion-workshop and comment the groups you don't want to create: 
  
    ```bash 
    cd aws-setup
    ansible-playbook provision-workshop.yml"
    ```
    
 9. We now have a bunch of servers, each with their unique tags. Now we need configure the servers of each group so 
    that they can find each other by writing the access key and appropriate tag info in the inventories of each group's 
    buildserver . Wait a while for cloud-init to complete (takes approx 2 minutes, for certainty check the 
    `/var/log/cloud-init.log` on a buildserver). Then fill in the AmazonEC2ReadOnlyAccess account credentials and execute: 
 
    ```bash
    # We must use the workshop private key to connect to the servers, this is the only way in!
    ssh-agent bash
    ssh-add workshop_ansiblecc_key

    ansible-playbook -i ../ansible/production configure-inventories-for-each-group.yml -e "ansible_read_access_key=<READONLY_KEY> ansible_read_secret_key=<READONLY_SECRET>"
    ```
    
 10. Provide each group with the IP addresses of their servers, the pipes strip away unnecessary info:
 
    ```bash
    ansible-playbook -i ../ansible/production list-all-workshop-ip-addresses.yml | grep msg | sort | cut -c 13- | sed 's/.$//' | tee /tmp/ip.txt
    ```

     