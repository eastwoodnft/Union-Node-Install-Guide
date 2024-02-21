# Union-Node-Install-Guide
#If running your own hardware:
1. Download Debian
2. Create .iso image using balena/rufus
3. install Debian, uncheck GNOME and Desktop Env (no need)
4. install dependencies
   sudo apt -q update
   sudo apt -qy install curl git jq lz4 build-essential
   sudo apt -qy upgrade

  #install go
   sudo rm -rf /usr/local/go
   curl -Ls https://go.dev/dl/go1.22.0.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
   eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
   eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)

  #install docker
    #Pre Requisites
     sudo apt update
     sudo apt install apt-transport-https ca-certificates curl software-properties-common

    #Download deb files
     curl https://download.docker.com/linux/debian/dists/bookworm/pool/stable/amd64/containerd.io_1.6.28-1_amd64.deb -o ~/dockerfiles/containerd.io_1.6.28-1_amd64.deb
     curl https://download.docker.com/linux/debian/dists/bookworm/pool/stable/amd64/docker-ce-cli_25.0.3-1~debian.12~bookworm_amd64.deb -o ~/dockerfiles/docker-ce-cli_25.0.3-1~debian.12~bookworm_amd64.deb
     curl https://download.docker.com/linux/debian/dists/bookworm/pool/stable/amd64/docker-buildx-plugin_0.12.1-1~debian.12~bookworm_amd64.deb -o ~/dockerfiles/docker-buildx-plugin_0.12.1-1~debian.12~bookworm_amd64.deb
     curl https://download.docker.com/linux/debian/dists/bookworm/pool/stable/amd64/docker-ce_25.0.3-1~debian.12~bookworm_amd64.deb -o ~/dockerfiles/docker-ce_25.0.3-1~debian.12~bookworm_amd64.deb
     curl https://download.docker.com/linux/debian/dists/bookworm/pool/stable/amd64/docker-compose-plugin_2.24.5-1~debian.12~bookworm_amd64.deb -o ~/dockerfiles/docker-compose-plugin_2.24.5-1~debian.12~bookworm_amd64.deb

    #Install packages
     sudo dpkg -i ~/dockerfiles/./containerd.io_1.6.28-1_amd64.deb \
    ~/dockerfiles/./docker-ce_25.0.3-1~debian.12~bookworm_amd64.deb \
    ~/dockerfiles/./docker-ce-cli_25.0.3-1~debian.12~bookworm_amd64.deb \
    ~/dockerfiles/./docker-buildx-plugin_0.12.1-1~debian.12~bookworm_amd64.deb \
    ~/dockerfiles/./docker-compose-plugin_2.24.5-1~debian.12~bookworm_amd64.deb

    #Verify it's running
     sudo docker run hello-world

    #Set docker at sudo
     sudo groupadd docker
     sudo usermod -aG docker $USER
     newgrp docker
     docker run hello-world


# Setup Environment varioables
export UNIOND_VERSION=v0.19.0
export CHAIN_ID=union-testnet-6
export MONIKER="eastwood"
export KEY_NAME=wallet
export GENESIS_URL="union.testnet.6.seed.poisonphang.com:26657/genesis"

# Download project binaries
mkdir -p $HOME/.union/cosmovisor/genesis/bin
wget -O $HOME/.union/cosmovisor/genesis/bin/uniond https://snapshots.kjnodes.com/union-testnet/uniond-v0.19.0-linux-amd64
chmod +x $HOME/.union/cosmovisor/genesis/bin/uniond

# Create application symlinks
sudo ln -s $HOME/.union/cosmovisor/genesis $HOME/.union/cosmovisor/current -f
sudo ln -s $HOME/.union/cosmovisor/current/bin/uniond /usr/local/bin/uniond -f


8. Pull and Run Docker
   docker pull ghcr.io/unionlabs/uniond-release:$UNIOND_VERSION
   sudo touch compose.yaml
   sudo nano compose.yaml
   COMPOSE_FILE=~/compose.yaml docker compose up -d

curl -Ls $GENESIS_URL > $HOME/.union/config/genesis.json
   curl $GENESIS_URL | jq '.result.genesis' > ~/.union/config/genesis.json

   sed -i -e "s|^seeds *=.*|seeds = \"b37de4c50e26f7cde4c7b6ce06046a6693ffef2c@union.testnet.6.seed.poisonphang.com:26656\"|" ~/.union/config/config.toml
   sed -i -e "s|^seeds *=.*|seeds = \"3f472746f46493309650e5a033076689996c8881@union-testnet.rpc.kjnodes.com:17159\"|" $HOME/.union/config/config.toml
