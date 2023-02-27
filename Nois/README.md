# Nois testnet guide V003

![nnnoi](https://user-images.githubusercontent.com/44331529/191945004-1227fef0-a215-44f1-bcab-854acd66de00.png)

[Website](https://nois.network/)
=
[EXPLORER 1](http://explorer.stavr.tech/nois/staking) \
[EXPLORER 2](https://testnet.ping.pub/nois/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 160GB    |


# 1) Auto_install script
```Python
wget -O nois https://raw.githubusercontent.com/obajay/nodes-Guides/main/Nois/nois && chmod +x nois && ./nois
```

# 2) Manual installation

### Preparing the server

```Python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 19 

```Python
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 07.10.22 
```Python
cd ~
git clone https://github.com/noislabs/full-node.git
cd full-node/full-node/
git checkout nois-testnet-003
./build.sh
mv out/noisd /usr/local/bin
```
`noisd version`
- 0.29.0-rc2

```Python
noisd init STAVRguide --chain-id nois-testnet-003
```    

## Create/recover wallet
```Python
noisd keys add <walletname>
noisd keys add <walletname> --recover
```

## Download Genesis

```Python
cd $HOME/.noisd/config/
rm genesis.json
curl -O https://raw.githubusercontent.com/noislabs/testnets/main/nois-testnet-003/genesis.json
```
`sha256sum $HOME/.noisd/config/genesis.json`
+ 9153084f305111e72fed86f44f6a11711c421532722200c870170d98223233ba

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```Python
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.noisd/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.noisd/config/config.toml
peers="2bf8002d0f65c3d86fca31ea0f043d912682c3e0@65.109.70.23:17356"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.noisd/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.noisd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.noisd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.noisd/config/config.toml
export CONFIG_DIR="$HOME/.noisd/config"
# Update app.toml
sed -i 's/minimum-gas-prices =.*$/minimum-gas-prices = "0.05unois"/' $CONFIG_DIR/app.toml
# Update config.toml
sed -i 's/^timeout_propose =.*$/timeout_propose = "2000ms"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_propose_delta =.*$/timeout_propose_delta = "500ms"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_prevote =.*$/timeout_prevote = "1s"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_prevote_delta =.*$/timeout_prevote_delta = "500ms"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_precommit =.*$/timeout_precommit = "1s"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_precommit_delta =.*$/timeout_precommit_delta = "500ms"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_commit =.*$/timeout_commit = "1800ms"/' $CONFIG_DIR/config.toml
```
### Pruning (optional)
```Python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.noisd/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.noisd/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.noisd/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.noisd/config/app.toml
```
### Indexer (optional) 
```Python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.noisd/config/config.toml
```

## Download addrbook
```Python
wget -O $HOME/.noisd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Nois/addrbook.json"
```

# StateSync
```Python
SNAP_RPC=http://nois.rpc.t.stavr.tech:21037
peers="2dc7ab934dfec910fac3083fd74e3451e1d3e670@135.181.5.47:21036"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.noisd/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.noisd/config/config.toml
noisd tendermint unsafe-reset-all --home $HOME/.noisd --keep-addr-book
systemctl restart noisd && journalctl -u noisd -f -o cat
```

# Snaphot 09.10.22 (0.1 GB) block height --> 46921
```Python
# install the node as standard, but do not launch. Then we delete the .data directory and create an empty directory
sudo systemctl stop noisd
rm -rf $HOME/.noisd/data/
mkdir $HOME/.noisd/data/
# download archive
cd $HOME
wget http://nois.snap.stavr.tech:7050/noisddata.tar.gz
# unpack the archive
tar -C $HOME/ -zxvf noisddata.tar.gz --strip-components 1
# !! IMPORTANT POINT. If the validator was created earlier. Need to reset priv_validator_state.json  !!
wget -O $HOME/.noisd/data/priv_validator_state.json "https://raw.githubusercontent.com/obajay/StateSync-snapshots/main/priv_validator_state.json"
cd && cat .noisd/data/priv_validator_state.json

# after unpacking, run the node
# don't forget to delete the archive to save space
cd $HOME
rm noisddata.tar.gz
systemctl restart noisd && journalctl -u noisd -f -o cat
```


# Create a service file
```Python
sudo tee /etc/systemd/system/noisd.service > /dev/null <<EOF
[Unit]
Description=noisd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which noisd) start --home $HOME/.noisd
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```Python
sudo systemctl daemon-reload
sudo systemctl enable noisd
sudo systemctl restart noisd && sudo journalctl -u noisd -f -o cat
```

### Create validator
```Python
noisd tx staking create-validator \
--amount=99000000unois \
--pubkey=$(noisd tendermint show-validator) \
--moniker=STAVRguide \
--chain-id=nois-testnet-003 \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.01" \
--min-self-delegation "1" \
--identity="" \
--details="" \
--website="" \
--from=<wallet> \
--fees=16000unois \
--gas=300000 \
-y
```

## Delete node
```Python
sudo systemctl stop noisd && \
sudo systemctl disable noisd && \
rm /etc/systemd/system/noisd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf full-node && \
rm -rf .noisd && \
rm -rf $(which noisd)
```

