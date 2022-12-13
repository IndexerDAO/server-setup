# Server Setup - Launchpad Part 1
## Prerequisites
### Configure host machine 
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

### Configure client machine
Remove stale host fingerprints from `~/.ssh/known_hosts`

``` bash
nano ~/.ssh/known_hosts
ctrl + k
ctrl + x
y
```

Install Taskfile
* Releases can be found here: https://github.com/go-task/task/releases

``` bash
cd ~
wget https://github.com/go-task/task/releases/download/v3.17.0/task_linux_amd64.tar.gz
tar zxvf task_linux_amd64.tar.gz
sudo ln -s /home/alex/task /usr/local/bin # change depending on your download path
```

### Create a hosted source code version control repository
We use GitHub: https://github.com/IndexerDAO/launchpad-office-hours
* Don't create a `README.md`, `LICENSE`, or `.gitignore`

## Launchpad
Clone Launchpad-Starter to your client device

``` bash
cd code/IndexerDAO/LOH
git clone https://github.com/graphops/launchpad-starter launchpad-office-hours
cd launchpad-office-hours
git remote remove origin
```

Commit files to your hosted source code version control repo

``` bash
git remote add origin https://github.com/IndexerDAO/launchpad-office-hours.git
git push origin main
```

Install launchpad-core submodule, commit changes to git, and push to GitHub

``` bash
sudo task launchpad:setup
git add .
git commit -m "feat: added launchpad-core submodule"
git push origin main
```

Update `inventory/inventory.yaml` with our host IP, port, and username using [`single_node.sample.yaml`](https://github.com/graphops/launchpad-starter/blob/main/inventory/samples/single-node.sample.yaml) template

``` bash
todo
```

Bootstraps host with Kubernetes

``` bash
task hosts:apply-base -- -e ansible_ssh_port=22 -e ansible_user=root
task hosts:apply-k0s
mkdir -p ~/.kube
mv ~/.kube/config ~/.kube/config.backup.$(date +%s)
cp inventory/artifacts/k0s-kubeconfig.yml ~/.kube/config
chmod 600 ~/.kube/config
```

Install non-Graph components of our stack

``` bash
task releases:apply-base
```
