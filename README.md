# Haqq Network - MainNet


## Overview
The current version of the HAQQ MainNet is [`v1.6.0`](https://github.com/haqq-network/haqq/releases/tag/v1.6.0). To bootstrap a mainnet node, use State Sync and synchronize a snapshot from our official seed nodes.


## Quickstart
_*Battle tested on [Ubuntu LTS 22.04](https://spinupwp.com/doc/what-does-lts-mean-ubuntu/#:~:text=The%20abbreviation%20stands%20for%20Long,extended%20period%20over%20regular%20releases)*_

**You can do the same yourself**

Install packages:
```sh
sudo apt-get update && \
sudo apt-get install curl git make gcc liblz4-tool build-essential git-lfs jq -y
```

**Preresquisites for compile from source**
- `make` & `gcc` 
- `Go 1.20+` ([How to install Go](https://www.digitalocean.com/community/tutorials/how-to-install-go-on-ubuntu-20-04))

**Easy GO compiler and HAQQ node installation**

```sh
curl -OL https://raw.githubusercontent.com/haqq-network/mainnet/master/install_go.sh && \
curl -OL https://raw.githubusercontent.com/haqq-network/mainnet/master/install_haqq.sh && \
sh install_go.sh && \ 
source $HOME/.bash_profile && \
sh install_haqq.sh
```

**Do the same manually:**

Download latest binary for your arch: </br>
https://github.com/haqq-network/haqq/releases/tag/v1.6.0

Build from source:
```sh
cd $HOME
git clone -b v1.6.0 https://github.com/haqq-network/haqq
cd haqq
git lfs fetch --all
make install
```

Verify binary version:
```sh
haqq@haqq-node:~# haqqd -v
haqqd version "1.6.0" e87024c28f935cd84e142cca16620c4f1ca681e1
```

**Initialize and start HAQQ**

```sh
export CUSTOM_MONIKER="mainnet_seed_node"
export HAQQD_DIR="$HOME/.haqqd"

haqqd config chain-id haqq_11235-1 && \
haqqd init $CUSTOM_MONIKER --chain-id haqq_11235-1

# Prepare genesis file for mainet(haqq_11235-1)
curl -L https://raw.githubusercontent.com/haqq-network/mainnet/master/genesis.json -o $HAQQD_DIR/config/genesis.json

# Prepare addrbook
curl -L https://raw.githubusercontent.com/haqq-network/mainnet/master/addrbook.json -o $HAQQD_DIR/config/addrbook.json

# Configure State sync
curl -OL https://raw.githubusercontent.com/haqq-network/mainnet/master/state_sync.sh && \
chmod +x state_sync.sh && \
./state_sync.sh $HAQQD_DIR

# Start Haqq
haqqd start
```

## Cosmovisor setup

1. Install cosmovisor bin
```sh
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@latest
```

2. Create cosmovisor folders
```sh
mkdir $HAQQD_DIR/cosmovisor && \
mkdir -p $HAQQD_DIR/cosmovisor/genesis/bin && \
mkdir -p $HAQQD_DIR/cosmovisor/upgrades
```

3. Copy node binary into Cosmovisor folder
```sh
cp /root/go/bin/haqqd $HAQQD_DIR/cosmovisor/genesis/bin
```

4. Create haqqd cosmovisor service
```sh
nano /etc/systemd/system/haqqd.service
```

```sh
[Unit]
Description="haqqd cosmovisor"
After=network-online.target

[Service]
User=root
ExecStart=/root/go/bin/cosmovisor run start
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment="DAEMON_NAME=haqqd"
Environment="DAEMON_HOME=$HAQQD_DIR"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=false"

[Install]
WantedBy=multi-user.target
```

5. Enable and start service

```sh
systemctl enable haqqd.service && \
systemctl start haqqd.service
```

6. Check logs
```sh
journalctl -fu haqqd
```

## Run in Docker

```sh
export CUSTOM_MONIKER="mainnet_seed_node"
export HAQQD_DIR="/root/haqqd_dock"
export HAQQD_VERSION="v1.6.0"

### Check it works
docker run -it --rm \
-v $HAQQD_DIR:/home/haqq/.haqqd \
alhaqq/haqq:$HAQQD_VERSION \
haqqd -v

### Init
docker run -it --rm \
-v $HAQQD_DIR:/home/haqq/.haqqd \
alhaqq/haqq:$HAQQD_VERSION \
haqqd init $CUSTOM_MONIKER --chain-id haqq_11235-1

curl -OL https://raw.githubusercontent.com/haqq-network/mainnet/master/genesis.json && \
mv genesis.json $HAQQD_DIR/config/genesis.json && \
curl -OL https://raw.githubusercontent.com/haqq-network/mainnet/master/addrbook.json && \
mv addrbook.json $HAQQD_DIR/config/addrbook.json && \
curl -OL https://raw.githubusercontent.com/haqq-network/mainnet/master/state_sync.sh && \
sh state_sync.sh $HAQQD_DIR

### Start
docker run -it \
--network host \
-v $HAQQD_DIR:/home/haqq/.haqqd \
alhaqq/haqq:$HAQQD_VERSION \
haqqd start
```
