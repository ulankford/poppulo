# poppulo project

Overview

This jumpbox has been setup with the following.
Ansible 2.9 installed
Nessesary python module installed e.g boto3
Key-pair nessesary to do a git clone from github.com (~/.ssh/id_rsa)
Key-pair nessesary to upload the public key to AWS used to ssh into our new EC2 Infrastructure (~/.ssh/pop-webapp)

1: ssh to jumpbox which also acts as an ansible control node
pop.pem key file will be attached in the email. Make sure to do a chmod 0600 pop.pem for it to work

`ssh -i pop.pem ec2-user@34.243.48.102`


2: Once on the jumpbox, as ec2-user do a git clone of this repo in the ec2-user home directory

`git clone git@github.com:ulankford/poppulo.git`

3: Now we have you repo, we can now call Ansible to create our nessesary platform
The following tasks done here are:
    Upload a public key to AWS (this will be used to log into our new instance)
    Create a new AWS Security Group with nessesary rules needed to connect to the instance and use our app
    Create our new AWS EC2 instance
    Write the new Public IP address of the AWS to a hosts file (this will be used by the next playbook to connect to it and run some plays)
    Wait a period of time to make sure the SSH service of our new AWS Instance is up and running
    Finally, we build our webapp by taking the index.jsp file and creating a Java WAR file which will be imported to our Tomcat Server
    
`ansible-playbook -i ~/poppulo/ansible/hosts ~/poppulo/ansible/deploy.yml --vault-password-file ~/poppulo/ansible/group_vars/all/vaultfile`

4: The final step is to deploy the application to the newly created ec2 aws instance. 
The following tasks done here are:
    Enable the nessesary repos to install the docker rpm
    Install and Start Docker
    Copy over the newly built WAR and the Dockerfile
    Build our new Docker Image (we use a Tomcat Docker Image from docker.io)
    Run our new Image as a Docker container listening on host port 80

`ansible-playbook -i ~/poppulo/ansible/hosts ~/poppulo/ansible/install.yml --vault-password-file ~/poppulo/ansible/group_vars/all/vaultfile`

5: Test that this work

A number of ways to test this. 
First you need the public IP, which can be found on the jumpbox under
`cat ~/poppulo/ansible/hosts`
The public IP of the EC2 instance will be under the [aws_poppulo_webapp] ansible group 
