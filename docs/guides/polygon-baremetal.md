# Polygon Node Setup Instructions
#### Configure Hetzner AX101 server
Put server into RescueMode from https://robot.hetzner.com/server
* Click server link then “Rescue”
* Note down the new password
    * Will need to update bitwarden with new password
* Go to “Reset” then select “Execute an automatic hardware reset” 
    * Now you can access the server with username=root and the password from step b.

<br>

Install Ubuntu on machine using Hetzner installimage utility
* Type `installimage` in RescueMode CLI then press enter
* Select `Ubuntu` then select `Ubuntu-2204-jammy-amd64-base.tar.gz`
* From within the config file:
    * Update `SWRAIDLEVEL` to 0
    * Update `HOSTNAME` to Polygon-Node
    * Update `RAID` configs as follows:
        * `PART swap swap 8G`
        * `PART /boot ext3 512M`
        * `PART / ext4 all`
* Type `F10` > select `Save changes`
* After install is complete, reboot the device
* Ssh into the server and verify that available space is greater than 7 terabytes
    * `df -h --total`

<br>
<br>

#### Install prerequisites

``` bash
apt update && apt upgrade
sudo apt install -y build-essential bsdmainutils aria2 golang dtrx screen clang cmake curl httpie jq nano wget
go install github.com/a8m/envsubst/cmd/envsubst@latest
sudo apt install docker.io docker-compose
```

<br>
<br>

#### Download Erigon snapshot

``` bash
screen -S erigon
ERIGON_HOME=$HOME/.local/share/erigon/datadir
mkdir -p ${ERIGON_HOME}
SNAPSHOT_URL=https://matic-blockchain-snapshots.s3-accelerate.amazonaws.com/matic-mainnet/erigon-archive-snapshot-2023-01-12.tar.gz
wget --tries=0 -O - "${SNAPSHOT_URL}" | tar -xz -C ${ERIGON_HOME} && touch ${ERIGON_HOME}/bootstrapped
screen -X detach 
```

<br>
<br>

#### Download Heimdall snapshot

``` bash
screen -S heimdall
HEIMDALL_HOME=$HOME/.local/share/heimdall/data
mkdir -p ${HEIMDALL_HOME}
SNAPSHOT_URL=https://matic-blockchain-snapshots.s3-accelerate.amazonaws.com/matic-mainnet/heimdall-snapshot-2023-01-17.tar.gz
wget --tries=0 -O - "${SNAPSHOT_URL}" | tar -xz -C ${HEIMDALL_HOME} && touch ${HEIMDALL_HOME}/bootstrapped
screen -X detach 
```

<br>
<br>

#### Clone Erigon and install

``` bash
git clone -b v0.0.5 https://github.com/maticnetwork/erigon
cd erigon
make
```

<br>
<br>

#### Clone Heimdall and install

``` bash
cd ~
git clone -b v0.3.0 https://github.com/maticnetwork/heimdall
cd heimdall
make build network=mainnet
```

<br>
<br>

#### Configure Heimdall

``` bash
$HOME/heimdall/build/heimdalld init --home $HOME/.local/share/heimdall/
wget -O $HOME/.local/share/heimdall/config/genesis.json https://raw.githubusercontent.com/maticnetwork/launch/master/mainnet-v1/without-sentry/heimdall/config/genesis.json
sed -i '/^seeds/c\seeds = "f4f605d60b8ffaaf15240564e58a81103510631c@159.203.9.164:26656,4fb1bc820088764a564d4f66bba1963d47d82329@44.232.55.71:26656,2eadba4be3ce47ac8db0a3538cb923b57b41c927@35.199.4.13:26656,3b23b20017a6f348d329c102ddc0088f0a10a444@35.221.13.28:26656,25f5f65a09c56e9f1d2d90618aa70cd358aa68da@35.230.116.151:26656"' $HOME/.local/share/heimdall/config/config.toml
sed -i "s#^cors_allowed_origins.*#cors_allowed_origins = [\"*\"]#" $HOME/.local/share/heimdall/config/config.toml
```

<br>
<br>

#### Create systemd files for Erigon, Heimdalld and Heimdallr
`Erigon` service file

``` bash
sudo echo "[Unit]
Description=Erigon Polygon Service
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=$HOME/.local/share/erigon/
ExecStart=$HOME/erigon/build/bin/erigon --chain="bor-mainnet" --datadir="$HOME/.local/share/erigon/datadir" --ethash.dagdir="$HOME/.local/share/erigon/datadir/ethash" --snapshots="true" --bor.heimdall="http://localhost:1317" --http --http.addr="0.0.0.0" --http.port="8545" --http.compression --http.vhosts="*" --http.corsdomain="*" --http.api="eth,debug,net,trace,web3,erigon,bor" --ws --ws.compression --rpc.gascap="300000000" --metrics --metrics.addr="0.0.0.0" --metrics.port="6969"
KillSignal=SIGHUP

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/erigon.service
```

<br>

`Heimdalld` service file

``` bash
sudo echo "[Unit]
Description=Heimdalld
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=$HOME/.local/share/heimdall/
ExecStart=$HOME/heimdall/build/heimdalld --home $HOME/.local/share/heimdall/ start
KillSignal=SIGHUP

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/heimdalld.service
```

<br>

`Heimdallr` service file

``` bash
sudo echo "[Unit]
Description=Heimdallr
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/.local/share/heimdall/
ExecStart=/root/heimdall/build/heimdalld --home /root/.local/share/heimdall/ rest-server --chain-id=137
KillSignal=SIGHUP

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/heimdallr.service
```

<br>
<br>

#### Start heimdalld and heimdallr services

``` bash
sudo systemctl daemon-reload
sudo systemctl enable heimdalld 
sudo systemctl enable heimdallr
sudo systemctl start heimdalld heimdallr
```

<br>
<br>

#### Check Heimdall status

``` bash
sudo journalctl -fu heimdalld
curl http://localhost:26657/status
```

<br>
<br>

#### Start Erigon

``` bash
#ONLY AFTER heimdall caught up to the chainhead, i.e. "catching_up": false in `curl http://localhost:26657/status` response
sudo systemctl enable erigon
sudo systemctl start erigon
```

<br>
<br>

#### Check Erigon status

``` bash
sudo journalctl -fu erigon
```

<br>
<br>

#### Resources

* [MIPS Polygon Phase 1 Notion Page](https://thegraphfoundation.notion.site/Polygon-Phase-1-Serving-Polygon-Subgraphs-on-The-Graph-s-Goerli-Testnet-0baf495b4442494b96afe3d0c3864e38)
* [Payne’s Polygon Node installation guide](https://thegraphfoundation.notion.site/Polygon-RPC-using-Erigon-77a651bd46544df5b59ed49f17289f7e)
* [Payne’s Polygon snapshot download helper script](https://github.com/cventastic/POKT_DOKT/blob/main/polygon/erigon/scripts/entrypoint.sh)

