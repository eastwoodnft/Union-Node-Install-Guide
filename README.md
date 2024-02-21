# Union-Node-Install-Guide for Ubuntu 22.04.3 LTS
>Using information from KJ Nodes with minor changes to get rid of errors

## Install dependencies
```
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential
sudo apt -qy upgrade
```

## Setup Environment varioables
```
export UNIOND_VERSION=v0.19.0
export CHAIN_ID=union-testnet-6
export MONIKER="eastwood" #edit to call it whatever you want your nodes moniker to be
export KEY_NAME=wallet
```

## Install GO
```
sudo rm -rf /usr/local/go
curl -Ls https://go.dev/dl/go1.22.0.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)
```

## Download project binaries
```
mkdir -p $HOME/.union/cosmovisor/genesis/bin
wget -O $HOME/.union/cosmovisor/genesis/bin/uniond https://snapshots.kjnodes.com/union-testnet/uniond-v0.19.0-linux-amd64
chmod +x $HOME/.union/cosmovisor/genesis/bin/uniond
```

## Create application symlinks
```
sudo ln -s $HOME/.union/cosmovisor/genesis $HOME/.union/cosmovisor/current -f
sudo ln -s $HOME/.union/cosmovisor/current/bin/uniond /usr/local/bin/uniond -f
```

## Download and install Cosmovisor
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```
## Create service
```
sudo tee /etc/systemd/system/union.service > /dev/null << EOF
[Unit]
Description=union node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --home=$HOME/.union
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.union"
Environment="DAEMON_NAME=uniond"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.union/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable union.service
```

# Workaround mandatory home argument
```
alias uniond='uniond --home=$HOME/.union/'
```

# Initialize the node
```
uniond init $MONIKER --chain-id union-testnet-6 --home=$HOME/.union
```

# Download genesis file
```
curl -Ls https://snapshots.kjnodes.com/union-testnet/genesis.json > $HOME/.union/config/genesis.json
```

# Add seeds
```
sed -i -e "s|^seeds *=.*|seeds = \"3f472746f46493309650e5a033076689996c8881@union-testnet.rpc.kjnodes.com:17159\"|" $HOME/.union/config/config.toml
```

# Set minimum gas price
```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0muno\"|" $HOME/.union/config/app.toml
```

# Set pruning (optional)
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.union/config/app.toml
```
# Start Service
```
sudo systemctl start union.service
```
# Check Logs & Syncinfo
```
sudo journalctl -u union.service -f --no-hostname -o cat
uniond status | jq
```
