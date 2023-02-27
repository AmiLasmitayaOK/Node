# Prepare
- Detile Event : https://nibiru.fi/blog/posts/007-itn-1.html
- Check explorer ID di https://nibiru.explorers.guru/ 
- Register : https://gleam.io/yW6Ho/nibiru-incentivized-testnet-registration
__________________________________

# Auto Install
```
wget -O nibiru.sh https://raw.githubusercontent.com/ArumaSanjayani/Node/main/Nibiru%20Chain/nibiru.sh && chmod +x nibiru.sh && ./nibiru.sh
```
## Check Log dan Sync pasca Install

### check logs
```
sudo journalctl -u nibid -f --no-hostname -o cat
```

### Chek sync log setelah 10 menit
```
nibid status 2>&1 | jq .SyncInfo
```
### check version
```
nibid version
```

##### Kalau Pake Auto Install, setelah kelar, langsung lompat ke bagian add wallet gan
__________________________________

# Manual Install
Check di : xxxx
__________________________________
# Add Wallet

### Jika Membuat Wallet baru
```
nibid keys add wallet
```

### Jika Import Wallet yang sudah ada
```
nibid keys add wallet --recover
```

### Untuk melihat daftar wallet saat ini
```
nibid keys list
```
### Simpan Info Wallet
```
NIBIRU_WALLET_ADDRESS=$(nibid keys show $WALLET -a)
NIBIRU_VALOPER_ADDRESS=$(nibid keys show $WALLET --bech val -a)
echo 'export NIBIRU_WALLET_ADDRESS='${NIBIRU_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export NIBIRU_VALOPER_ADDRESS='${NIBIRU_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
### Faucet 
Discord : https://discord.gg/2J5jw9se4A

### Check Saldo Wallet
```
nibid query bank balances $NIBIRU_WALLET_ADDRESS
```

### Restart Node
```
sudo systemctl daemon-reload
sudo systemctl enable nibid
sudo systemctl restart nibid
source $HOME/.bash_profile
```
__________________________________
# Validator management
### Create Validator
```
nibid tx staking create-validator \
--amount 1000000unibi \
--pubkey $(nibid tendermint show-validator) \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id nibiru-itn-1 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.025unibi \
-y
```

### Jika ingin Edit existing validator
```
nibid tx staking edit-validator \
--new-moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL"
--chain-id nibiru-itn-1 \
--commission-rate 0.05 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.025unibi \
-y
```

__________________________________


# Uninstall Node
```
sudo systemctl stop nibid
sudo systemctl disable nibid
sudo rm /etc/systemd/system/nibid.service
sudo systemctl daemon-reload
rm -f $(which nibid)
rm -rf $HOME/.nibiru
rm -rf $HOME/nibiru
rm -rf $HOME/nibiru.sh
rm -rf $HOME/go
```

__________________________________

Unjail validator
```
nibid tx slashing unjail --from wallet --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

Jail reason
```
nibid query slashing signing-info $(nibid tendermint show-validator)
```

List all active validators
```
nibid q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

List all inactive validators
```
nibid q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

View validator details
```
nibid q staking validator $(nibid keys show wallet --bech val -a)
```

💲 Token management
Withdraw rewards from all validators
```
nibid tx distribution withdraw-all-rewards --from wallet --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

Withdraw commission and rewards from your validator
```
nibid tx distribution withdraw-rewards $(nibid keys show wallet --bech val -a) --commission --from wallet --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

Delegate tokens to yourself
```
nibid tx staking delegate $(nibid keys show wallet --bech val -a) 1000000unibi --from wallet --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

Delegate tokens to validator
```
nibid tx staking delegate <TO_VALOPER_ADDRESS> 1000000unibi --from wallet --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

Redelegate tokens to another validator
```
nibid tx staking redelegate $(nibid keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 1000000unibi --from wallet --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

Unbond tokens from your validator
```
nibid tx staking unbond $(nibid keys show wallet --bech val -a) 1000000unibi --from wallet --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

Send tokens to the wallet
```
nibid tx bank send wallet <TO_WALLET_ADDRESS> 1000000unibi --from wallet --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```


🗳 Governance
List all proposals
```
nibid query gov proposals
```

View proposal by id
```
nibid query gov proposal 1
```

Vote 'Yes'
```
nibid tx gov vote 1 yes --from wallet --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

Vote 'No'
```
nibid tx gov vote 1 no --from wallet --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

Vote 'Abstain'
```
nibid tx gov vote 1 abstain --from wallet --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

Vote 'NoWithVeto'
```
nibid tx gov vote 1 nowithveto --from wallet --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

⚡️ Utility
Update ports
```
CUSTOM_PORT=10
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}660\"%" $HOME/.nibid/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}317\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}091\"%" $HOME/.nibid/config/app.toml
```

Update Indexer
Disable indexer
```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.nibid/config/config.toml
```

Enable indexer
```
sed -i -e 's|^indexer *=.*|indexer = "kv"|' $HOME/.nibid/config/config.toml
```

Update pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.nibid/config/app.toml
```

🚨 Maintenance
Get validator info
```
nibid status 2>&1 | jq .ValidatorInfo
```

Get sync info
```
nibid status 2>&1 | jq .SyncInfo
```

Get node peer
```
echo $(nibid tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.nibid/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

Check if validator key is correct
```
[[ $(nibid q staking validator $(nibid keys show wallet --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(nibid status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Get live peers
```
curl -sS http://localhost:39657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

Set minimum gas price
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025unibi\"/" $HOME/.nibid/config/app.toml
```

Enable prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.nibid/config/config.toml
```

Reset chain data
```
nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book
```

⚙️ Service Management
Reload service configuration
```
sudo systemctl daemon-reload
```

Enable service
```
sudo systemctl enable nibid
```

Disable service
```
sudo systemctl disable nibid
```

Start service
```
sudo systemctl start nibid
```

Stop service
```
sudo systemctl stop nibid
```

Restart service
```
sudo systemctl restart nibid
```

Check service status
```
sudo systemctl status nibid
```

Check service logs
```
sudo journalctl -u nibid -f --no-hostname -o cat
```

__________________________________
### Source

https://services.kjnodes.com/home/testnet/nibiru/useful-commands

