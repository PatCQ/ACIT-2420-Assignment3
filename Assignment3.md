Welcome to a tutorial about configuring a fresh Debian 12 server on digitalocean.
This tutorial will show you how to:
    - create a new user with approaiate administartive access 
    - lock out the root user
    - Install nginx
        - configure nginx to showcase a sample website

This guide assumes that you have access and can connect to your Debian droplet server

# Step 1: Connect to the Fresh Debian 12 server
From the directory .ssh (c:\Users\Username\.ssh)
Enter the command in the console 

```powershell
ssh -i do-key root@ip_address_of_the_server
```

# Step 2: Create a new user
While connected to the Debian server,
Enter the command:

```bash
useradd -ms /bin/bash username
```
*replace username with the name of the new user*

    This will create a new user with the name username while also creating a home directory for the new user and sets the login shell to /bin/bash

# Step 3: Set a password for the new user
Enter the command:
    
```bash
passwd username
```

It will prompt you to enter a "New password:"
    Enter the desired password to be associated with the new user
        The password you entered will be invisible for security purposes
After entering the password once you will be prompted to "Retype new Password:"
    Enter the password again. Make sure the password is exactly the same

# Step 4: Grant access to new user access to preform adminstrative tasks
Enter the command:

```bash
usermod -aG sudo username
```

this command is case sensitive make sure that in "-aG" the G is capitalized

by calling the command usermod with -aG we ensure that the group is added without removing any other groups the user is a part of. 

# Step 5: Give new user access to connect to server via ssh
Enter the command:

```bash
cp -r .ssh /home/username
```

This will copy the directory .ssh and all it's contents to the desired directory. We do this to give the new user the .ssh file and it's authorized ssh keys so that it can connect via ssh to the server

Enter the command:

```bash
chown -R username:username /home/username/.ssh
```

This will overwrite the user and group that owns the .ssh directory and its files to the new user

We can now test if this connection works or not by exiting the root user and then connecting back with the new user

```bash
exit
```

```powershell
ssh -i do-key newuser@ip_address_of_the_server
```

# Step 6: Removing root user access to the server via ssh
Connect back to the root user 
Use the following command if you are connected to the new user

```bash
exit
```


Connect to the root

```conosle
ssh -i do-key root@143.198.229.76
```

Access the sshd_config file:

```bash
vim /etc/ssh/sshd_config
```


In vim look for the line "PermitRootLogin yes" and change yes to no
in visual mode, press / and type PermitRootLogin
Press enter and press n to find the uncommented line of "PermitRootLogin yes" and change from yes to no

Save the changes and exit with the command :wq

With the changes saved we will now restart the ssh.service

```bash
systemctl restart ssh.service
```

Now we can test if root can still connect via ssh or not
    
```bash
exit
```

```powershell
ssh -i do-key root@ip_address_of_the_server
```   

An error should show up saying:
"root@ip_address_of_the_server: Permission denied (publickey)"

# Step 7: Install nginx
Connect to the new user

```powershell
ssh -i do-key newuser@ip_address_of_the_server
```

We need to update apt
Run the command:
    
```bash
sudo apt update
```

You may need to enter the user's password as this might be the first time you called sudo in this session.
Enter the password and the command will be executed

Now we can install nginx
Run the command:
    
```bash
sudo apt install nginx
```

There may be a confirmation confirming that it is OK to use additional disk space. 
    Type: Y
    and type enter to continue the operation

# Step 8: Configuring nginx to serve a smaple website
Create a directory to house the html of the website
Run the command:

```bash
sudo mkdir /var/www/sample-site
```

Create the main html file of the website
Run the command:

```bash
sudo vim /var/www/sample-site/index.html
```

Enter the following content into index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420</title>
    <style>
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
        }
    </style>
</head>
<body>
    <h1>Hello, World</h1>
</body>
</html>
```
Make sure to save the file with :wq in vim

Create the configuration file inside of /etc/nginx/sites-available/
Run the command:

```bash
sudo vim /etc/nginx/sites-available/sample-site.conf
```

within the file enter the following content:
```
server {
	listen 80 default_server;
	listen [::]:80 default_server;
	
	root /var/www/sample-site;
	
	index index.html index.htm index.nginx-debian.html;
	
	server_name _;
	
	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}
}
```

Create a symbolic link of the conf file from sites-available to sites-enabled
Run the command:

```bash
sudo ln -s /etc/nginx/sites-available/sample-site.conf /etc/nginx/sites-enabled/sample-site.conf
```

Remove the deafult symbolic link

```bash
sudo unlink /etc/nginx/sites-enabled/default
```

We can now test nginx configurations with the following line:

```bash
sudo nginx -t
``` 

If all is well, it should output two lines:
nginx: the configuration file /etc/nginx/nginx/conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

the first line shows that the configuration file is properlly configured.
the second line shows that the symbolic link was correctly made.

If there are no errors then we can restart the nginx service
Enter the commmand:
```bash
sudo systemctl restart nginx
```

Using servers ip address, the address we use to connect to the server.
We can check the sample html with the following commmand

```bash
curl ip_address
```
This should output the html content that was entered into index.html earlier.

