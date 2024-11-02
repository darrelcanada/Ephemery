## Validator install using Lighthouse Client

 Create a service user for the validator service and create data directories.

```bash
sudo adduser --system --no-create-home --group validator
sudo mkdir -p /var/lib/lighthouse/validators
```

Import your validator keys by importing your keystore file. Be sure to enter your keystore password correctly.

```bash
sudo lighthouse account validator import \
  --testnet-dir /etc/ephemery \
  --datadir /var/lib/lighthouse \
  --directory=$HOME/Downloads/ethstaker_deposit-cli-66054f5-linux-amd64/validator_keys \
  --reuse-password
```

Verify that your keystore file was imported successfully.

```bash
sudo lighthouse account_manager validator list \
  --testnet-dir /etc/ephemery \
  --datadir /var/lib/lighthouse
```

Setup ownership permissions, including hardening the access to this directory.

```bash
sudo chown -R validator:validator /var/lib/lighthouse/validators
sudo chmod 700 /var/lib/lighthouse/validators
```

Create a systemd unit file to define your ```validator.service``` configuration.

```bash
sudo nano /etc/systemd/system/validator.service
```

Paste the following configuration into the file. 

```bash
[Unit]
Description=Lighthouse Validator Client service for Ephermery
Wants=network-online.target
After=network-online.target
Documentation=https:https://github.com/darrelcanada/Ephemery

[Service]
Type=simple
User=validator
Group=validator
Restart=on-failure
RestartSec=3
KillSignal=SIGINT
TimeoutStopSec=900
EnvironmentFile=/etc/ephemery/nodevars_env.txt
ExecStart=/usr/local/bin/lighthouse vc \
  --testnet-dir /etc/ephemery \
  --beacon-nodes http://localhost:5052 \
  --datadir /var/lib/lighthouse \
  --graffiti="By: Rocket Pool" \
  --metrics \
  --metrics-port 8009 \
  --suggested-fee-recipient=0x108B5d9A49F8275c1194D07920627A28a55bC7e4

[Install]
WantedBy=multi-user.target
```
