
# poppulo project

*Overview of solution*

The hello-world web app consists of an index.jsp file that will be served by a docker container running Tomcat.
The container itself will be running on a EC2 AWS instance that has Docker installed and runnnig as a daemon.
index.jsp file can be found in /roles/build_app/files/hello/index.jsp on this repo

This jumpbox has been setup with the following:
 - Ansible 2.9 installed
 - Nessesary python modules installed e.g boto3
 - Key-pair nessesary to do a git clone from github.com ( ~/.ssh/id_rsa )
 - Key-pair nessesary to upload the public key to AWS used to ssh into our new EC2 instance ( ~/.ssh/pop-webapp )


*Deploy Steps*
 1. ssh to jumpbox which acts as an ansible control node.
The pop.pem key file will be attached in the email which will be used to log into this server. Make sure to do a chmod 0600 pop.pem for it to work if using a terminal or cli

`ssh -i pop.pem ec2-user@34.243.48.102`


2. Once logged onto the jumpbox, as the ec2-user do a git clone of my repo in the ec2-user home directory.

`git clone git@github.com:ulankford/poppulo.git`


3. Now we have you the repo cloned, we can now call Ansible playbooks to create our nessesary platform.

`ansible-playbook -i ~/poppulo/ansible/hosts ~/poppulo/ansible/deploy.yml --vault-password-file ~/poppulo/ansible/group_vars/all/vaultfile`

The following tasks done here are:

  - Upload a public key to AWS (this will be used to log into our new instance)
  - Create a new AWS Security Group with nessesary rules needed to ssh to the server and serve http requests
  - Create our new AWS EC2 instance based on a RHEL 7 image
  - Write the new Public IP address of the new AWS EC2 instance to a hosts file (this will be used by the follwing playbook to ssh to his new instance and run some plays)
  - Wait a period of time to make sure the SSH service of our new AWS Instance is up and running
  - Finally, we build our webapp by taking the index.jsp file and creating a Java WAR file which will be imported to our Tomcat Docker image
    



4. The final step is to deploy the application to the newly created ec2 aws instance. 

`ansible-playbook -i ~/poppulo/ansible/hosts ~/poppulo/ansible/install.yml --vault-password-file ~/poppulo/ansible/group_vars/all/vaultfile`

The following tasks done here are:
   - Enable the nessesary repos to install the docker rpm
   - Install and Start Docker
   - Copy over the newly built WAR and the Dockerfile
   - Build our new Docker Image (we use a Tomcat Docker Image from docker.io)
   - Run our new Image as a Docker container listening on host port 80




5. Test that one can connect to the new web app and it serves you the correct page

 - First you need the public IP, which can be found on the jumpbox under:
 
   `cat ~/poppulo/ansible/hosts`
   
   The public IP of the EC2 instance will be under the [aws_poppulo_webapp] ansible group

 - Use the Public IP address and get your browser to point to it
 
   `http://< Public_IP_of_Instance >/hello`
   
  - Alternatively, you can use a linux tools like curl or elinks ( these are installed on the jumpbox already for your convenience ).
  
   `curl http://< Public_IP_of_Instance  >/hello/`
   
   `elinks http://< Public_IP_of_Instance  >/hello`
   
   Note the trailing / in the curl command, with this trailing / tomcat returns a 302 redirection message
  
 - Lastly, you can connect to the Docker container itself and view the tomcat logs
 
   `ssh -i ~/.ssh/pop-webapp < Public_IP_of_Instance  >` 
   
   `sudo docker ps` ( This will output the Container ID )
   
   `sudo docker exec -it < docker_container_id > /bin/bash`
   
   `cat logs/localhost_access_log* `
 
  
