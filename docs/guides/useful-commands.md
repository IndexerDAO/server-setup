# Useful commands

Show currently running services
* `systemctl --type=service`

Delete stale ssh fingerprint
* `ssh-keygen -R [IP_ADDRESS]`

Run previous command as `sudo`
* `sudo !!`

Create JSON Web Token (JWT)
* `sudo openssl rand -hex 32 | sudo tee /var/lib/jwtsecretgnosis/jwtgnosis.hex > /dev/null`

Download Lighthouse 3.3.0
* `curl -LO https://github.com/sigp/lighthouse/releases/download/v3.3.0/lighthouse-v3.3.0-x86_64-unknown-linux-gnu.tar.gz`

Extract Lighthouse
* `tar vvf lighthouse-v3.3.0-x86_64-unknown-linux-gnu.tar.gz`

Rename lighthouse folder
* `mv lighthouse lighthousegnosis`

Copy lighthouse folder to /usr/local/bin
* `sudo cp lighthousegnosis /usr/local/bin`

Remove unnecessary tar.gz file for housekeeping
* `sudo rm lighthouse-v3.3.0-x86_64-unknown-linux-gnu.tar.gz`

Remove unnecessary directory for housekeeping
* `sudo rm lighthousegnosis`

Create directory
* `sudo mkdir -p /var/lib/lighthousegnosis/beacon`

Update permission of directory to lighthousegnosis user
* `sudo chown -R lighthousegnosis:lighthousegnosis /var/lib/lighthousegnosis/beacon`

Open ports
* `sudo ufw allow 9901/tcp`
* `sudo ufw allow 9001/udp`

Enable and start nethermind and lighthousegnosis services
* `sudo systemctl daemon-reload`
* `sudo systemctl enable lighthousegnosis`
* `sudo systemctl start lighthousegnosis`
* `sudo systemctl enable nethermind`
* `sudo systemctl start nethermind`

Delete NVME partitions
``` bash
delete_partitions() {
 if [ "$1" ]; then
  # clean RAID information for every partition not only for the blockdevice
  for raidmember in $(sfdisk -l "$1" | grep -o "${1}p\?[0-9]\+"); do
    mdadm -q --zero-superblock "$raidmember" 
  done
  # clean RAID information in superblock of blockdevice
  mdadm -q --zero-superblock "$1" 

  # delete GPT and MBR
  sgdisk -Z "$1" 

  # clean mbr boot code
  dd if=/dev/zero of="$1" bs=512 count=1 status=none ; EXITCODE=$?

  # re-read partition table
  partprobe 

  return $EXITCODE
 fi
}

delete_partitions /dev/nvme0n1
delete_partitions /dev/nvme1n1

``` 
