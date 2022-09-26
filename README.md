# Server Setup - Gnosis Nethermind 1.14.1


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

`sudo useradd --no-create-home --shell /bin/false nethermind` Creates a user named `nethermind`

  + `--shell /bin/false`  is just a binary that immediately exits, returning false, when it's called, so when someone has `false`
 as the shell logs in, they're immediately logged out.
 
 <br><br>
 
## UFW - Uncomplicated firewall 
  
`sudo apt install ufw` Install firewall

`sudo ufw allow 22` ssh

`sudo ufw allow 40403` nethermind peers

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
## If UsePAM is enabled, you will not be able to run sshd(8) as a non-root user. The default is “no”. 
```
  
<br><br>

  
## Gnosis Nethermind
    
`curl -LO [https://github.com/NethermindEth/nethermind/releases/download/1.14.1/nethermind-linux-amd64-1.14.1-1a32d45-20220907.zip](https://github.com/NethermindEth/nethermind/releases/download/1.14.1/nethermind-linux-amd64-1.14.1-1a32d45-20220907.zip)` Downloads a .zip file with the nethermind binaries
    
`unzip nethermind-linux-amd64-1.14.1-1a32d45-20220907.zip -d nethermind` Unzips the zip file we just downloaded into a folder called nethermind
    
  + `-d` Specify which directory to unzip the files
    
`sudo cp -a nethermind /usr/local/bin/nethermind` Copies nethermind into /usr/local/bin/
    
  + `-a` Copies the file with the same permission settings and metadata as the original.
    
`rm nethermind-linux-amd64-1.14.1-1a32d45-20220907.zip` Removes the zip file
    
`rm -r nethermind` Removes the unzipped nethermind folder (we already copied this to  /usr/local/bin/)
    
`sudo apt-get update && sudo apt-get install libsnappy-dev libc6-dev libc6 unzip` Downloads and installs dependencies
    
`sudo mkdir -p /var/lib/nethermind` Makes a directory for the datafiles
    
`sudo chown -R nethermind:nethermind /var/lib/nethermind` Changes the owner of the newly created data folder to the nethermind user
    
`sudo nano /etc/systemd/system/nethermind.service` Edit the nethermind service file
    
```graphql
[Unit]
Description=Nethermind Execution Client (Gnosis)
After=network.target
Wants=network.target
[Service]
User=nethermind
Group=nethermind
Type=simple
Restart=always
LimitNOFILE=1000000
RestartSec=5
WorkingDirectory=/var/lib/nethermind
Environment="DOTNET_BUNDLE_EXTRACT_BASE_DIR=/var/lib/nethermind"
ExecStart=/usr/local/bin/nethermind/Nethermind.Runner \
  --config gnosis_archive_rpc \
  --datadir /var/lib/nethermind \
  --Metrics.Enabled true \
  --Network.P2PPort 40403 \
  --Network.DiscoveryPort 40403
[Install]
WantedBy=default.target
```
    
`sudo nano /usr/local/bin/nethermind/configs/gnosis_archive_rpc.cfg` Edit the configuration file
    
```jsx
{
  "Init": {
    "ChainSpecPath": "chainspec/xdai.json",
    "GenesisHash": "0x4f1dd23188aab3a76b463e4af801b52b1248ef073c648cbdc4c9333d3da79756",
    "BaseDbPath": "nethermind_db/xdai_archive",
    "LogFileName": "xdai_archive.logs.txt",
    "MemoryHint": 1024000000
  },
  "Mining": {
    "MinGasPrice": "1000000000"
  },
  "EthStats": {
    "Name": "Nethermind xDai"
  },
  "Metrics": {
    "NodeName": "xDai Archive"
  },
  "Bloom":
  {
    "IndexLevelBucketSizes" : [16, 16, 16]
  },
  "Pruning": {
    "Mode": "None"
  },
  "JsonRpc": {
    "Enabled": true,
    "Timeout": 20000,
    "Host": "127.0.0.1",
    "Port": 9545,
    "EnabledModules": [
      "Eth",
      "AccountAbstraction",
      "Trace",
      "TxPool",
      "Web3",
      "Personal",
      "Proof",
      "Net",
      "Parity",
      "Health",
      "Rpc"
    ]
  }
}
```
    
`sudo systemctl daemon-reload`
    
`sudo systemctl start nethermind`
    
`sudo systemctl enable nethermind`
    
<br><br>
  
Logs:
    
`sudo journalctl -fu nethermind`
   
