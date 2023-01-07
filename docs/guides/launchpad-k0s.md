# Launchpad Setup: k0s
## Prerequisites

### Configure host machine 

Put Hetzner AX101 server into RescueMode from https://robot.hetzner.com/server

* Click server link then “Rescue”
    * Note down the new password
    * Will need to update bitwarden with new password
* Go to “Reset” then select “Execute an automatic hardware reset”
* Wait a few minutes then you can access the server with username=root and the password from step b.


Install [Ubuntu 22.04](https://releases.ubuntu.com/22.04/) on [Hetzner AX101 dedicated server](https://www.hetzner.com/dedicated-rootserver/ax101) with [`installimage`](https://docs.hetzner.com/robot/dedicated-server/operating-systems/installimage/) script

* Type installimage in RescueMode CLI then press enter
* Select Ubuntu then select Ubuntu-2204-jammy-amd64-base.tar.gz
* From within the config file:
    * Update SWRAIDLEVEL to 0
    * Update HOSTNAME to Polygon-Node
    * Update RAID configs as follows:
        * PART swap swap 8G
        * PART /boot ext3 512M
        * PART / ext4 all
* Type F10 > select Save changes
* After install is complete, reboot the device
* Ssh into the server and verify that available space is greater than 7 terabytes
    * `df -h --total`


Set up [ssh key pair authentication](https://help.ubuntu.com/community/SSH/OpenSSH/Keys)
``` bash
# create an ssh keypair if you don't already have one ready: run `ssh-keygen -t rsa` on client machine
nano ~/.ssh/authorized_keys
# paste your public ssh key from client machine into the file on host
ctrl + x
y
# optional: use Termius export to host functionality instead of copy + paste to host
```

### Configure client machine
Remove any [stale fingerprints](https://en.wikipedia.org/wiki/Public_key_fingerprint) the host(s) may have added to client `~/.ssh/known_hosts`

``` bash
nano ~/.ssh/known_hosts
ctrl + k
ctrl + x
y
```

Install [Taskfile](https://github.com/go-task/task/releases)

``` bash
cd ~
wget https://github.com/go-task/task/releases/download/v3.17.0/task_linux_amd64.tar.gz
tar zxvf task_linux_amd64.tar.gz
sudo ln -s /home/alex/task /usr/local/bin # change depending on your download path
```

### Create a hosted source code version control repository
We use GitHub: https://github.com/IndexerDAO/launchpad-office-hours
> Note: Don't create a `README.md`, `LICENSE`, or `.gitignore`

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
# Example of an inventory for single host that will be configured
# as both the Kubernetes master and as a Kubernetes worker node

all:
  vars:
    hardened_ssh_port: &hardened-ssh-port 1500
  hosts:
    launchpad: # you can customise the name of your host
      ansible_host: 162.55.134.32

init_group:
  hosts:
    launchpad:
  vars:
    ansible_user: root
    hardened_ssh_port: *hardened-ssh-port
    main_user: &main-user paka
    enable_sshd_config: true # change to true to enable ssh hardening and lock port 22
    enable_lvm_config: false

k0s:
  vars:
    ansible_user: *main-user
    become: true
    k0s_version: v1.24.6+k0s.0
    k0s_use_custom_config: false
    ansible_ssh_port: *hardened-ssh-port
  children:
    initial_controller:
      hosts:
        launchpad:
      vars:
        # --no-taints removes node-role.kubernetes.io/master taint
        # so that k8s workloads can be provisioned on the controller node
        # konnectivity-server is not required in a one node flat network cluster
        extra_args: "--enable-worker --no-taints --disable-components konnectivity-server"
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
