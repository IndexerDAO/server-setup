# server-setup


## Install Ubuntu
`installimage` From rescue. Install Ubuntu

`Ubuntu` 

`Ubuntu-2004-focal-64-minimal`

```jsx
SWRAIDLEVEL 0
...

HOSTNAME <Our Name>
...

PART swap swap 128G
PART /boot ext3 1024M
PART / ext4 all
```

`F10` → `yes` → `yes`

`reboot`

<br><br>

Useful commands:

`cat /proc/mdstat` Raid Status

`df -h` Check free space on disc

<br><br>
## Server setup
`apt update && apt upgrade` Updates packages

`apt install build-essential` Install essentials

`apt install git`

`apt install unzip`

<br><br>
## User Management

`adduser <username>` Adds a new user. 

`usermod -aG sudo <username>` Adds the user to the “sudo” group, giving them access to run “sudo” commands

  + `-aG` assign a user to one or more supplementary groups 

<details>

<summary>Add ssh keys directly</summary>

<br>
    
`su <username>` switch to other user
    
`cd ~` `cd <username>` Enters the home dir of the user we just created
    
`mkdir .ssh` creates “.ssh” 
    
`touch .ssh/authorized_keys` creates “.ssh/autorized_keys”
    
`chmod 700 .ssh` Set permissions for the .ssh folder
    
`chmod 600 .ssh/authorized_keys` Set permissions for the authorized key file
    
`cd .ssh` opens the “.ssh” folder
    
`nano authorized_keys`
    
Add Public keys - one key per line

<br>

</details>
    
<details>

<summary> OR Add ssh keys with termius </summary>

<br>

export keys with termius

<br>
    
</details>
    

`sudo useradd --no-create-home --shell /bin/false lighthousebeacon` Creates a user named `lighthousebeacon`

`sudo useradd --no-create-home --shell /bin/false erigon` Creates a user named `erigon`

`sudo useradd --no-create-home --shell /bin/false nethermind` Creates a user named `nethermind`

  + `--shell /bin/false`  is just a binary that immediately exits, returning false, when it's called, so when someone has `false`
 as the shell logs in, they're immediately logged out.
 
 <br><br>
 
## UFW - Uncomplicated firewall 
  
`sudo apt install ufw` Install firewall

`sudo ufw allow 22` ssh

`sudo ufw allow 30303` erigon peers

`sudo ufw allow 40303` nethermind peers

`sudo ufw allow 9000/tcp` lighthouse peers

`sudo ufw allow 9000/udp` lighthouse peers

`sudo ufw enable` Enables ufw

 <br><br>
  
Useful commands:

`ufw status` Prints status

`ufw disable` Disables ufw
  
 <br><br>
  
 ## SSH
 
 ! WARNING ! Do not proceed with this step until you confirm being able to SSH into the new user you created. 
  
 sudo nano /etc/ssh/sshd_config
  
 ```PermitRootLogin no
## Prevents root logins through ssh

PasswordAuthentication no
## Can only access server through a password

UsePAM no
## If UsePAM is enabled, you will not be able to run sshd(8) as a non-root user. The default is “no”.```
  
<br><br>
  
## Go
  
`wget -c https://golang.org/dl/go1.19.1.linux-amd64.tar.gz` download go

`sudo tar -C /usr/local -xvzf go1.19.1.linux-amd64.tar.gz` unpack go

`rm go1.19.1.linux-amd64.tar.gz` download go

`nano ~/.profile` edits “~/.profile”

`export PATH=$PATH:/usr/local/go/bin` added to the end of file

`sudo ln -s /usr/local/go/bin/go /usr/local/bin/go` Make the executable available to everyone.

`source ~/.profile` Refresh profile
  
<br><br>
  
Useful commands:

`go version` check version
  
 <br><br>
  
