# Union

# Union
Union Node Installation Instructions </br>
### [Official documentation](https://union.build/docs/)

### System requirements: </br>
CPU: 4 Core </br>
RAM: 8 Gb </br>
SSD: 200 Gb </br>
OS: Ubuntu 20.04 LTS </br>

### Network configurations: </br>
Outbound - allow all traffic </br>
Inbound - open the following ports :</br>
1317 - REST </br>
26657 - TENDERMINT_RP </br>
26656 - Cosmos </br>
Running as a Provider? Add these specific ports: </br>
22221 - provider port </br>
22231 - provider port </br>
22241 - provider port </br>
    
# Manual configuration of the Lava node with Cosmovisor
Required packages installation </br>
```
sudo apt update
sudo apt upgrade -y
sudo apt install -y curl git jq lz4 build-essential
sudo apt install -y unzip logrotate git jq sed wget curl coreutils systemd
```

Go installation.
```
cd $HOME
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```

### Download and build binaries
```
cd $HOME && mkdir -p go/bin/
wget -O uniond https://testnet-files.itrocket.net/union/uniond
chmod +x uniond
mv uniond $HOME/go/bin/
```

# Initialize the node
```
uniond init "Your Node Name" --chain-id union-testnet-8
```

# Download genesis and addrbook files
```
curl -L https://snapshots-testnet.nodejumper.io/union-testnet/genesis.json > $HOME/.union/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/union-testnet/addrbook.json > $HOME/.union/config/addrbook.json
```

# Set seeds
```
sed -i -e 's|^seeds *=.*|seeds = "f1d2674dc111d99dae4638234c502f4a4aaf8270@union.testnet.4.val.poisonphang.com:2665"|' $HOME/.union/config/config.toml
```

# Set minimum gas price
```
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0025muno"|' $HOME/.union/config/app.toml
```

# Set pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.union/config/app.toml
```

# Change ports
```
sed -i -e "s%:1317%:24617%; s%:8080%:24680%; s%:9090%:24690%; s%:9091%:24691%; s%:8545%:24645%; s%:8546%:24646%; s%:6065%:24665%" $HOME/.union/config/app.toml
sed -i -e "s%:26658%:24658%; s%:26657%:24657%; s%:6060%:24660%; s%:26656%:24656%; s%:26660%:24661%" $HOME/.union/config/config.toml
```

# Download latest chain data snapshot
```
curl "https://snapshots-testnet.nodejumper.io/union-testnet/union-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.union"
```

# Create a service
```
sudo tee /etc/systemd/system/uniond.service > /dev/null << EOF
[Unit]
Description=Union node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which uniond) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable uniond.service
```

# Start the service and check the logs
```
sudo systemctl start uniond.service
sudo journalctl -u uniond.service -f --no-hostname -o cat
```

### Becoming a Validator

# Create wallet key new
```
uniond keys add wallet
```

(OPTIONAL) RECOVER EXISTING KEY
```
uniond keys add wallet --recover
```

### We receive tokens from the tap in the discord(https://discord.gg/union-build)

### Create the Validator

Before creating a validator, enter the command and check that you have false. This means that the Node has synchronized and you can create a validator:
```
uniond status 2>&1 | jq -r '.SyncInfo.catching_up // .sync_info.catching_up'
```

### Check the balance before creating for the presence of tokens
```
uniond q bank balances $(uniond keys show wallet -a)
```

Please enter your details below in quotation marks where required

```
uniond tx staking create-validator \
--amount=1000000muno \
--pubkey=$(uniond tendermint show-validator) \
--moniker="Your nick" \
--identity=FFB0AA51A2DF5955 \
--details="I love YTWO" \
--chain-id=union-testnet-6 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=0.0025muno \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

### Update
```
There have been no updates at the moment, as soon as they come out, we will immediately add them to this section.

Current network:union-testnet-8
Current version:v0.20.0
```

### Useful commands

Check balance
```
uniond q bank balances $(uniond keys show wallet -a)
```

CHECK SERVICE LOGS
```
sudo journalctl -u uniond -f --no-hostname -o cat
```

RESTART SERVICE
```
sudo systemctl restart uniond
```

GET VALIDATOR INFO
```
uniond status 2>&1 | jq -r '.ValidatorInfo // .validator_info'
```

DELEGATE TOKENS TO YOURSELF
```
uniond tx staking delegate $(uniond keys show wallet --bech val -a) 1000000muno --from wallet --chain-id union-testnet-6 --gas-prices 0.0025muno --gas-adjustment 1.5 --gas auto -y
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop uniond && sudo systemctl disable uniond && sudo rm /etc/systemd/system/uniond.service && sudo systemctl daemon-reload && rm -rf $HOME/.union  && sudo rm -rf $(which uniond)
```
