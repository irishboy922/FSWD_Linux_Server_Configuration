# Udacity's Linux Server Configuration Project

We were given a baseline installation of a Ubuntu Linux distribution on a virtual machine and asked to configure it in order to host a Flask web application. Some of the things that needed to be done include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

## Server Information
1. IP Address:
``` 
35.161.52.116
```
2. SSH Port:
``` 
2200 
```

## Web Application URL
```
http://35.161.52.116 
```

## Create Development Environment
(Resource: [Udacity](https://www.udacity.com/account#!/development_environment))

1. Create a new development environment based on the instructions Udacity provided in the project prep section.

2. Download the private key Udacity provided.

3. Move the private key to the folder ` ~/.ssh/ `:<br> 
` $ mv ~/Downloads/udacity_key.rsa ~/.ssh/ `

4. Change the modification permissions:<br>
` $ chmod 600 ~/.ssh/udacity_key.rsa ` 

5. Type in ` $ ssh -i ~/.ssh/udacity_key.rsa root@35.161.52.116 ` to access the terminal of the vm.

## Configuration Details
(Resource: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)) <br>

1. Create new user named grader: <br>
` $ sudo adduser grader `

2. Grant **grader** user sudo permissions:<br> 
Open configuration file:<br>
` $ sudo visudo `<br>
Add  the following code below ` root  ALL=(ALL:ALL) ALL `:<br>
` grader ALL=(ALL:ALL) ALL `<br>

3. Update and Upgrade packages:<br>
Update package list<br>
` $ sudo apt-get update `<br>
Upgrade packages<br>
` $ sudo apt-get upgrade `<br>

4. Configure the local time to UTC:<br>
(Resource: [Ask Ubuntu](http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)<br>
` $ sudo dpkg-reconfigure tzdata `<br>
Select **Other** from the menu then select **UTC** and press enter.

5. Change SHH port to 2200:<br>
(Resource: [Ask Ubuntu](http://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server))<br>
  - Open the SSH configuration file<br>
  ` $ sudo nano /etc/ssh/sshd_config `<br>
  - In the config file change port 20 to ` 2200 `<br>
  - Change the ` PermitRootLogin ` to ` no `<br>
  - Add ` AllowUsers grader ` to the file to allow **grader** user to login through SSH<br> 
  - Restart SSH<br>
  ` $ sudo service ssh restart `<br>

6. Create SSH Keys<br>
(Resource: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server))<br>
On local machine generate a SSH key pair by typing:<br>
` $ ssh-keygen `<br>
Open and copy the public key on local machine:<br>
` $ cat ~/.ssh/id_rsa.pub `<br>
Login to SSH<br>
` $ ssh grader@35.161.52.116 `<br>
Create a `.ssh` directory<br>
` $ mkdir .ssh `<br>
Create an `authorized_keys` file to store the public key<br>
` $ touch .ssh/authorized_keys `<br>
Open the `authorized_keys` file and paste the public key<br>
` $ nano .ssh/authorized_keys `<br>
Change the permissions on the `.ssh` folder and `authorized_keys` file<br>
` $ chmod 700 .ssh `<br>
` $ chmod 644 .ssh/authorized_keys `<br>
Disable
