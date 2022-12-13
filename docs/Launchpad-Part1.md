# Prerequisites
## Configure host machine 
Install Ubuntu 22.04

``` bash
installimage -n slimdexor -r no -i images/Ubuntu-2204-jammy-amd64-base.tar.gz -d nvme0n1,nvme1n1 -p /boot:ext3:512M,lvm:vg0:all -v vg0:root:/:ext4:all
```

Set up ssh key pair authentication
``` bash
nano ~/.ssh/authorized_keys
<!-- paste your public ssh key into the file  -->
ctrl + x
y
```

## Configure client machine
Remove stale host fingerprints from `~/.ssh/known_hosts`

``` bash
nano ~/.ssh/known_hosts
ctrl + k
ctrl + x
y
```

Install Taskfile
``` bash
todo
```

## Create source code version control repository
We use GitHub: https://github.com/IndexerDAO/launchpad-office-hours
* don't create a `README.md`, `LICENSE`, or `.gitignore`

# Getting Started
