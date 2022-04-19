Setup SFTP Server Ubuntu
==============

Install SSH
-------------
```bash
sudo apt install ssh
```

Creating an SFTP Group
-------------

Instead of configuring the OpenSSH server for each user individually we will create a new group and add all our chrooted users to this group.
Run the following groupadd command to create the sftponly user group:

```bash
sudo groupadd sftponly
```

Adding Users to the SFTP Group
--------------
The next step is to add the users you want to restrict to the sftponly group.
If this is a new setup and the user doesn’t exist you can create a new user account by typing:

```bash
sudo useradd -g sftponly -s /bin/false -m -d /home/username username
```
- The -g sftponly option will add the user to the sftponly group.
- The -s /bin/false option sets the user’s login shell. By setting the login shell to /bin/false the user will not be able to login to the server via SSH.
- The -m -d /home/username options tells useradd to create the user home directory.

Set a strong password for the newly created user:

```bash
sudo passwd username
```

Otherwise if the user you want to restrict already exist, add the user to the sftponly group and change the user’s shell:

```bash
sudo usermod -G sftponly -s /bin/false username2
```

The user home directory must be owned by root and have 755 permissions :

```bash
sudo chown root: /home/username
sudo chmod 755 /home/username
```

Since the users home directories are owned by the root user, these users will no be able to create files and directories in their home directories. If there are no directories in the user’s home, you’ll need to create new directories to which the user will have full access. For example, you can create the following directories:

```bash
sudo mkdir /home/username/{public_html,uploads}
sudo chmod 755 /home/username/{public_html,uploads}
sudo chown username:sftponly /home/username/{public_html,uploads}
```
If a web application is using the user’s public_html directory as document root, these changes may lead to permissions issues. For example, if you are running WordPress you will need to create a PHP pool that will run as the user owning the files and add the webs erver to the sftponly group.


Configuring SSH
-----------------
SFTP is a subsystem of SSH and supports all SSH authentication mechanisms.

Open the SSH configuration file /etc/ssh/sshd_config with your text editor :

```bash
sudo vim /etc/ssh/sshd_config
```
Search for the line starting with Subsystem sftp, usually at the end of the file. If the line starts with a hash # remove the hash # and modify it to look like the following:

```bash
  Subsystem sftp internal-sftp
```

Towards the end of the file, the following block of settings:

```bash
Match Group sftponly
  ChrootDirectory %h
  ForceCommand internal-sftp
  PasswordAuthentication yes
  PermitTunnel no
  AllowTcpForwarding no
  X11Forwarding no
```

The ChrootDirectory directive specifies the path to the chroot directory. %h means the user home directory. This directory, must be owned by the root user and not writable by any other user or group.

Be extra careful when modifying the SSH configuration file. The incorrect configuration may cause the SSH service to fail to start.

Once you are done save the file and restart the SSH service to apply the changes:

```bash
sudo systemctl restart ssh
```
In CentOS and Fedora the ssh service is named sshd:

```bash
sudo systemctl restart sshd
```

Testing the Configuration
Now that you have configured SFTP chroot you can try to login to the remote machine through SFTP using the credentials of the chrooted user. In most cases, you will use a desktop SFTP client like FileZilla but in this example, we will use the sftp command .

Open an SFTP connection using the sftp command followed by the remote server username and the server IP address or domain name:

```bash
sftp username@192.168.121.30
```
You will be prompted to enter the user password. Once connected, the remote server will display a confirmation message and the sftp> prompt:


```bash
username@192.168.121.30's password:
sftp>
```
Run the pwd command, as shown below, and if everything is working as expected the command should return /.

```bash
sftp> pwd
```
You can also list the remote files and directories using the ls command and you should see the directories that we have previously created:

```bash
sftp> ls
public_html  uploads  
```
