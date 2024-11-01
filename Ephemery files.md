# Installing  ephemery testnet files 

Create Directory /etc/ephemery

```bash
sudo mkdir /etc/ephemery
cd /etc/ephemery
```

download via wget testnet-all.tar.gz (use correct Iteration number ie: 139)

```bash
sudo wget https://github.com/ephemery-testnet/ephemery-genesis/releases/download/ephemery-139/testnet-all.tar.gz
```

Then unpack the archive 

```bash
sudo tar xzvf testnet-all.tar.gz
```

Set Permissions for Ephermery files and directory

```bash
sudo chown root:root *
sudo chown root:root .
```

Show permissions for Ephemery config files

```bash
ls -l
```
Should show root permissions

```bash
-rw-r--r-- 1 root root    41514 Oct 24 18:34 besu.json
-rw-r--r-- 1 root root      315 Oct 24 18:34 boot_enode.txt
-rw-r--r-- 1 root root     1111 Oct 24 18:34 boot_enr.txt
-rw-r--r-- 1 root root     1121 Oct 24 18:34 boot_enr.yaml
drwxr-xr-x 2 root root     4096 Oct 24 18:34 bootnode-keys
-rw-r--r-- 1 root root      315 Oct 24 18:34 bootnode.txt
-rw-r--r-- 1 root root     1111 Oct 24 18:34 bootstrap_nodes.txt
-rw-r--r-- 1 root root     1121 Oct 24 18:34 bootstrap_nodes.yaml
-rw-r--r-- 1 root root    42888 Oct 24 18:34 chainspec.json
-rw-r--r-- 1 root root     4102 Oct 24 18:34 config.yaml
-rw-r--r-- 1 root root        2 Oct 24 18:34 deploy_block.txt
-rw-r--r-- 1 root root       67 Oct 24 18:34 deposit_contract_block_hash.txt
-rw-r--r-- 1 root root        2 Oct 24 18:34 deposit_contract_block.txt
-rw-r--r-- 1 root root       43 Oct 24 18:34 deposit_contract.txt
-rw-r--r-- 1 root root      315 Oct 24 18:34 enodes.txt
-rw-r--r-- 1 root root    41507 Oct 24 18:34 genesis.json
-rwxr-xr-x 1 root root  5475676 Oct 24 18:34 genesis.ssz
-rw-r--r-- 1 root root       67 Oct 24 18:34 genesis_validators_root.txt
-rw-r--r-- 1 root root     2140 Oct 24 18:34 nodevars_env.txt
-rw-r--r-- 1 root root 16376315 Oct 24 18:34 parsedBeaconState.json
-rw-r--r-- 1 root root      175 Oct 24 18:34 retention.vars
-rw-r--r-- 1 root root  2980957 Oct 24 18:34 testnet-all.tar.gz
-rw-r--r-- 1 root root      413 Oct 24 18:34 validator-names.yaml
```





# Installing Nethermind Client

## 1. Initial configuration
Create a service user for the execution service, create data directory and assign ownership.

```bash
sudo adduser --system --no-create-home --group execution
sudo mkdir -p /var/lib/nethermind
sudo chown -R execution:execution /var/lib/nethermind
```

Install dependencies.

```bash
sudo apt update
sudo apt install ccze curl libsnappy-dev libc6-dev jq libc6 unzip -y
```

## 2. Install Binaries

Run the following to automatically download the latest linux release, un-zip and cleanup.

```bash
RELEASE_URL="https://api.github.com/repos/NethermindEth/nethermind/releases/latest"
BINARIES_URL="$(curl -s $RELEASE_URL | jq -r ".assets[] | select(.name) | .browser_download_url" | grep linux-x64)"

echo Downloading URL: $BINARIES_URL

cd $HOME
wget -O nethermind.zip $BINARIES_URL
unzip -o nethermind.zip -d $HOME/nethermind
rm nethermind.zip
```

Install the binaries.

```bash
sudo mv $HOME/nethermind /usr/local/bin/nethermind
```

## 3. Setup and configure systemd for Nethermind

```bash
sudo nano /etc/systemd/system/execution.service
```

```bash
[Service]
Type=simple
User=execution
Group=execution
Restart=always
RestartSec=3
KillSignal=SIGINT
TimeoutStopSec=900
WorkingDirectory=/var/lib/nethermind
Environment="DOTNET_BUNDLE_EXTRACT_BASE_DIR=/var/lib/nethermind"
EnvironmentFile=/etc/ephemery/nodevars_env.txt
ExecStart=/usr/local/bin/nethermind/nethermind \
  --config none.cfg \
  --datadir="/var/lib/nethermind" \
  --Network.DiscoveryPort 30303 \
  --Network.P2PPort 30303 \
  --Network.MaxActivePeers 50 \
  --JsonRpc.Port 8545 \
  --JsonRpc.EnginePort 8551 \
  --Metrics.Enabled true \
  --Metrics.ExposePort 6060 \
  --JsonRpc.JwtSecretFile /secrets/jwtsecret \
  --Pruning.Mode=Hybrid \
  --Pruning.FullPruningTrigger=VolumeFreeSpace \
  --Pruning.FullPruningThresholdMb=300000 \
  --JsonRpc.Enabled true \
  --Discovery.Bootnodes=${BOOTNODE_ENODE_LIST} \
  --Init.ChainSpecPath /etc/ephemery/chainspec.json \
  --Init.GenesisHash ${GENESIS_BLOCK}
[Install]
WantedBy=multi-user.target
```
To exit and save, press Ctrl + X, then Y, then Enter.


Run the following to enable auto-start at boot time.

```bash
sudo systemctl daemon-reload
sudo systemctl enable execution
```








# Installing Lighthouse Client

## 1. Initial configuration
Create a service user for the consensus service, create data directory and assign ownership.

```bash
sudo adduser --system --no-create-home --group consensus
sudo mkdir -p /var/lib/lighthouse
sudo chown -R consensus:consensus /var/lib/lighthouse
```

Install dependencies.

```bash
sudo apt install curl ccze jq -y
```

## 2. Install Binaries
Run the following to automatically download the latest linux release, un-tar and cleanup.

```bash
RELEASE_URL="https://api.github.com/repos/sigp/lighthouse/releases/latest"
BINARIES_URL="$(curl -s $RELEASE_URL | jq -r ".assets[] | select(.name) | .browser_download_url" | grep x86_64-unknown-linux-gnu.tar.gz$)"

echo Downloading URL: $BINARIES_URL

cd $HOME
# Download
wget -O lighthouse.tar.gz $BINARIES_URL
# Untar
tar -xzvf lighthouse.tar.gz -C $HOME
# Cleanup
rm lighthouse.tar.gz
```

Install the binaries.

```bash
sudo mv $HOME/lighthouse /usr/local/bin/lighthouse
```

## 3. Setup and configure systemd
Create a systemd unit file to define your consensus.service configuration.

```bash
sudo nano /etc/systemd/system/consensus.service
```

```bash
[Unit]
Description=Lighthouse Consensus Layer Client service for Holesky
Wants=network-online.target
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
Type=simple
User=consensus
Group=consensus
Restart=on-failure
RestartSec=3
KillSignal=SIGINT
TimeoutStopSec=900
EnvironmentFile=/etc/ephemery/nodevars_env.txt
ExecStart=/usr/local/bin/lighthouse bn \
  --datadir /var/lib/lighthouse \
  --testnet-dir /etc/ephemery \
  --staking \
  --validator-monitor-auto \
  --metrics \
  --port 9000 \
  --quic-port 9001 \
  --http-port 5052 \
  --target-peers 100 \
  --metrics-port 8008 \
  --execution-endpoint http://127.0.0.1:8551 \
  --execution-jwt /secrets/jwtsecret \
  --boot-nodes=${BOOTNODE_ENR_LIST}

[Install]
WantedBy=multi-user.target
```
To exit and save, press Ctrl + X, then Y, then Enter.

Run the following to enable auto-start at boot time.

```bash
sudo systemctl daemon-reload
sudo systemctl enable consensus
```

Start the services  

```bash
sudo systemctl start execution consensus
```

## 4. Helpful execution client commands
```bash
#View Logs:
sudo journalctl -fu execution | ccze  
#Stop:
sudo systemctl stop execution
#Start:
sudo systemctl start execution
#View Status:
sudo systemctl status execution
#Reset Database
sudo systemctl stop execution
sudo rm -rf /var/lib/nethermind/*
sudo systemctl restart execution
```

## Helpful consensus client commands

```bash
#View Logs:
sudo journalctl -fu consensus | ccze  
#Stop:
sudo systemctl stop consensus
#Start:
sudo systemctl start consensus
#View Status:
sudo systemctl status consensus
#Reset Database
sudo systemctl stop consensus
sudo rm -rf /var/lib/lighthouse/beacon
sudo systemctl restart consensus
```
<details>
<summary>Helpful consensus client commands</summary>

```bash
#View Logs:
sudo journalctl -fu consensus | ccze  
#Stop:
sudo systemctl stop consensus
#Start:
sudo systemctl start consensus
#View Status:
sudo systemctl status consensus
#Reset Database
sudo systemctl stop consensus
sudo rm -rf /var/lib/lighthouse/beacon
sudo systemctl restart consensus
```
<details>



<details>
<summary>Helpful execution client commands</summary>

```bash
#View Logs:
sudo journalctl -fu execution | ccze  
#Stop:
sudo systemctl stop execution
#Start:
sudo systemctl start execution
#View Status:
sudo systemctl status execution
#Reset Database
sudo systemctl stop execution
sudo rm -rf /var/lib/nethermind/*
sudo systemctl restart execution
```
</details>

