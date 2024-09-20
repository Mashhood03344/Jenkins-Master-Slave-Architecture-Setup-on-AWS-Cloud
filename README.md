# Jenkins Master-Slave Architecture Setup

## Table of Content

1. [Jenkins Master-Slave Architecture Setup](#jenkins-master-slave-architecture-setup)
2. [Why we need a Distributed Architecture](#why-we-need-a-distributed-architecture)
3. [Jenkins Distributed Architecture](#jenkins-disstributed-architecture)
 	- [Jenkins Master](#jenkins-msater)
 	- [Jenkins Slave](#jenkins-slave)
4. [Jenkins Master Slave Architecture Diagram](#jenkins-master-slave-architecture-diagram)
5. [How Jenkins Master and Slave Architecture works?](#how-jenkins-master-and-slave-architecture-works-?)
6. [Jenkins master-slave architecture Setup](#jenkins-master-slave-architecture-setup)
	- [Prerequisites](#prerequisites)
	- [Steps](#step)
	- [Trouble Shooting](#trouble-shooting)
7. [References](#references)


With the rise of micro services architecture, companies are increasingly required to develop and deploy multiple services simultaneously. To handle this complexity, CI tools play a critical role in distributing workloads across several machines or servers.
Jenkins is a leading CI tool with the ability to distribute build jobs across multiple systems, a feature known as Jenkins distributed builds. By setting up a network of build servers, Jenkins allows teams to efficiently spread the workload and execute build jobs in various environments, reducing bottlenecks and improving overall performance.

## Why we need a Distributed Architecture

In early computing, a single system (or server) was often responsible for handling all the processing tasks, data storage, and operations. As systems grew in complexity and the amount of data increased, one system couldn't keep up with the demand. This led to several issues:
1.	Overload and slow performance:
- A single machine or server would get overwhelmed when it had to process a lot of data or handle many user requests at once. This slowed down operations and led to bottlenecks (where the system became a point of delay).
2.	No fault tolerance:
- If that one system failed, the entire operation would come to a stop. There was no way to share or split the work among other systems to keep things running smoothly.
- Lack of scalability:
- As businesses or applications grew, it became harder to scale (increase capacity). Adding more users or processing more data required either upgrading the entire system (which was expensive) or buying bigger and more powerful hardware, which also had limits.

To solve these problems, Distributed architecture was proposed to distribute tasks across multiple systems. Here's how it addressed the issues:
- Load distribution: Instead of relying on one system, the master could delegate tasks to several slave systems, reducing the load on a single machine.
- Increased efficiency: With many slaves handling smaller tasks in parallel, operations became faster and more efficient.
- Fault tolerance: If one slave failed, the master could simply assign the task to another slave, ensuring the system continued working.
- Scalability: As demand grew, new slave systems could easily be added without major changes to the overall architecture, allowing for horizontal scaling (increasing capacity by adding more systems).
This architecture is widely adopted in systems that needed to handle large-scale data processing, distributed computing, or multi-tasking, such as:
- Databases (e.g., primary-replica setups)
- Distributed file systems (e.g., Hadoop)
- Network services (e.g., DNS or web servers)

## Jenkins Distributed Architecture

Jenkins uses A Master-Slave architecture to manage distributed builds. In Master-Slave Architecture, the master is like the boss who manages and controls everything, and slaves are like the workers who do the actual work. Master and Slave interact with each other using TCP/IP protocols.  This architecture helps us in such a way that if we have multiple processes running on Jenkins, it can easily manage them by distributing the processes to multiple other nodes (Slaves)

### Jenkins Master

The Jenkins Master is the central server responsible for controlling and managing the entire Jenkins environment. It handles critical tasks such as:
- Scheduling build jobs: The master schedules when and how build jobs will run.
- Dispatching builds to slaves: It assigns tasks to Jenkins slaves for execution, distributing the workload across multiple machines.
- Monitoring slaves: The master constantly monitors the state of slaves, taking them online or offline as needed.
- Recording and displaying results: It gathers build results from the slaves and presents them in the Jenkins dashboard.
Additionally, the Jenkins master can execute build jobs directly, but its primary role is to delegate tasks to slaves to optimize performance.

### Jenkins Slave

A Jenkins Slave is a remote machine that runs a Java executable and performs the actual execution of build jobs assigned by the master. Key features of Jenkins slaves include:
- Remote execution: Slaves are typically located on different machines (often with different operating systems) and can be distributed across multiple environments.
- Listening for requests: Slaves "listen" for instructions from the master and execute the assigned jobs accordingly.
- Configurable job allocation: You can configure specific jobs to always run on a particular slave or type of slave, or let Jenkins automatically assign jobs to the next available slave.
- Scalability: Slaves allow Jenkins to scale by offloading the execution of jobs to multiple machines, improving speed and efficiency in large builds or complex environments.

## Jenkins Master Slave Architecture Diagram

See the Jenkins Master and Slave Architecture.png

## How Jenkins Master and Slave Architecture works?

Now let us look at an example in which we use Jenkins for testing in different environments like Ubuntu, MAC, Windows, etc.
The diagram below represents the same:

Example Scenerio.png

The above image represents the following functions:

•	Jenkins checks the Git repository at periodic intervals for any changes made in the source code.
•	Each builds requires a different testing environment which is not possible for a single Jenkins server. In order to perform testing in different environments, Jenkins uses various Slaves as shown in the diagram.
•	Jenkins Master requests these Slaves to perform testing and to generate test reports.

## Jenkins master-slave architecture Setup

This guide outlines the process to set up a Jenkins master-slave architecture on two Ubuntu EC2 instances in the same VPC and subnet.

### Prerequisites

- Two EC2 instances (Master and Slave), both with Ubuntu AMI.
- Both instances should:
- Be in the same VPC and subnet.
- Have the same security group allowing SSH (port 22), HTTP (port 80), and Jenkins (port 8080).
- Share the same access keys.

### Steps

1. Connect to EC2 Instances

Use Instance Connect or SSH to access both instances.

2. Install Java and Jenkins on Both Instances

Update Package List

	sudo apt update

	Install OpenJDK 17
	
Install Java

	
	sudo apt install openjdk-17-jdk -y
	

Verify Java installation:

	
	java --version
	

Install Jenkins

Add the Jenkins repository and install:

	
	wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
	
	sudo sh -c 'echo deb http://pkg.jenkins.io/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
	
	sudo apt update
	
	sudo apt install jenkins -y
	


Start Jenkins and Enable It to Start on Boot

	
	sudo systemctl enable jenkins

	sudo systemctl start jenkins
	

Check Jenkins status:

	
	sudo systemctl status jenkins
	Retrieve Jenkins Initial Admin Password
	
The Initial Password for the jenkins

	
	sudo cat /var/lib/jenkins/secrets/initialAdminPassword
	

3. Configure SSH Key-Based Authentication

**On the Master Instance**

Generate an RSA key pair:

	
	ssh-keygen -t rsa -b 2048 -m PEM
	

Display the private key:

	
	cat ~/.ssh/id_rsa
	
	
Display the public key:

	
	cat ~/.ssh/id_rsa.pub
	

Copy the public key it would be like this ssh-rsa AAAAB3NzaC1yc2E...W+NbamljFnua6MB07/Hyz4Ep5n6naGvsu4B+m2tRZ.

**On the Slave Instance**

Create the necessary directories and add the master’s public key:

	
	sudo mkdir -p /home/ubuntu/.ssh

	sudo nano /home/ubuntu/.ssh/authorized_keys
	

Paste the copied public key into the authorized_keys file and save.

Change ownership of the .ssh directory:

	
	sudo chown -R ubuntu:ubuntu /home/ubuntu/.ssh
	

4. Create Jenkins Workspace on Slave

Create the Jenkins workspace directory:

	sudo mkdir -p /var/jenkins

	sudo chown ubuntu:ubuntu /var/jenkins

5. Test SSH Connection from Master to Slave

On the master instance, test the SSH connection to the slave:

	
	ssh ubuntu@<slave-instance-ip>
	

If the connection works, proceed to configure the slave in Jenkins.

6. Configure Jenkins Master to Use Slave

To createTo configure the node (or agent) settings on the master Jenkins server, follow these steps on the master server:

- Access Jenkins Dashboard
Open your Jenkins master server in a web browser: http://<master_ip>:8080.
Navigate to Manage Jenkins
On the Jenkins dashboard, click on "Manage Jenkins" in the left sidebar.
- Manage Nodes and Clouds
Click on "Manage Nodes and Clouds" (or "Manage Nodes" depending on your Jenkins version).
Add a New Node
Click on "New Node" or "New Node" in the sidebar.
- Enter Node Details
Node Name: Enter a name for your node.
Node Type: Select "Permanent Agent".
Click "OK".Initially, you will get only one option, "Permanent Agent." Once you have one or more slaves you will get the "Copy Existing Node" option.

- Configure Node Settings

You will now be on the configuration page for the new node. Fill out the following fields:

Remote root directory: The path on the slave machine where Jenkins will store files related to the build.
Labels: Add labels if you want to categorize this node.
Usage: Choose whether Jenkins should use this node for all builds or only those jobs that specifically require it.
Launch method: Choose how Jenkins should connect to this node:

Name: slave
Description: Slave node 
Number of executors: 2
Remote root directory: /var/jenkins
Labels :test
Usage: Use this node as much as possible
Launch method: Launch agents via SSH

Host: 3.83.122.37
Credentials: 

ubuntu

Host Key Verification Strategy: Non verifying Verification Strategy:

Availability: Keep this agent online as much as possible


Host: Enter the IP address or hostname of the slave instance (the slave instance ip address).
Credentials: Add the SSH credentials for the slave instance:
Click on "Add" next to "Credentials".
Choose "SSH Username with private key".
Enter the "Username":ubuntu.
Private Key: Select Enter directly and paste the content of your private key file (/home/ubuntu/.ssh/id_rsa). You can view the key with:
 
 on the master server and paster it in the field directly 
 
 
Copy the entire key, including the 
-----BEGIN RSA PRIVATE KEY-----MIIEpAIBAAKCAQEAv+TVTwRzGDAphdwxmmT+rlBcGChJtxMZm+ix1/yteh9KPgpS
BjPOQ+MCJ+qYt9cmdDRIU5EyWqURguAfkJ5/GCCS6OOSDueG+2kROKnNw6HJLZXP
jATgdK6MmilfakURKTE.....RRMQ3PUCgYBqjp175JUZ5l9l6KO+Dizw/mEN+Tp1tca8Wd2sLTdsUKwl8TLhBs7h
3zlui8csJf2PsQZmfJezfauSm/txtYzLOPh4Cb+nodSxPdOekeUQnHwYp2dAapWe
RcRqURktu700QATgx3ESerV0DOB18xK+mjN3AZRqegLebE/BA6u7gA==
-----END RSA PRIVATE KEY----- 
lines.


Click "OK" to add the credentials.
8. Save the Configuration
Click "Save" to apply the changes.
9. Verify Node Connectivity
Go back to "Manage Nodes and Clouds".
You should see the new node listed. Its status should be "Online" if everything is configured correctly.
Troubleshooting
If the node status is "Offline" or you encounter errors, check:
SSH connectivity: Ensure the Jenkins master can connect to the slave via SSH.
Firewall/Security Groups: Ensure proper network access is allowed.
Permissions: Make sure the Jenkins user has appropriate permissions on the slave machine.
Once configured, Jenkins will be able to use the slave node to run builds and jobs according to the configuration you've set.

### Trouble Shooting:

- Node Offline or SSH Connection Fails
SSH Connectivity:

Ensure the public key from the master is correctly copied to /home/ubuntu/.ssh/authorized_keys on the slave.
Verify correct permissions on the slave:

sudo chown -R ubuntu:ubuntu /home/ubuntu/.ssh


If you encounter an SSH connection issue, it is likely due to the public key from the master instance not being correctly copied or pasted into the slave instance. Ensure that the public key located at /home/ubuntu/.ssh/id_rsa.pub on the master instance has been accurately copied and added to the /home/ubuntu/.ssh/authorized_keys file on the slave instance.

Example public key:

ssh-rsa AAAAB3NzaC1yc2E...W+NbamljFnua6MB07/Hyz4Ep5n6naGvsu4B+m2tRZ

- Remoting.jar Not Found:

Ensure the /var/jenkins directory exists on the slave:

sudo mkdir -p /var/jenkins

sudo chown ubuntu:ubuntu /var/jenkins

- Firewall/Security Groups:

Ensure the security group allows traffic on SSH (port 22), HTTP (port 80), and Jenkins (port 8080).


## References

https://medium.com/@dksoni4530/jenkins-master-slave-architecture-setup-f0486fba8039
https://medium.com/edureka/jenkins-master-and-slave-architecture-e3d6c4728945
https://cloudzenia.com/blog/understanding-master-slave-architecture-in-jenkins/
https://aurigait.com/blog/jenkins-master-and-slave-nodes/
https://dzone.com/articles/jenkins-03-configure-master-and-slave



