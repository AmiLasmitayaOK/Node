# Nolus Snapshot
https://snapshot.nolus.deffanok.me/nolus/
______________________________
        
## How to Use our Snapshot
### Stop the service and reset the data
```
sudo systemctl stop nolusd
cp $HOME/.nolus/data/priv_validator_state.json $HOME/.nolus/priv_validator_state.json.backup
rm -rf $HOME/.nolus/data
```
### Download our snapshot
```
curl -L https://snapshot.nolus.deffanok.me/nolus/snapshot_latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.nolus
mv $HOME/.nolus/priv_validator_state.json.backup $HOME/.nolus/data/priv_validator_state.json
```

### Restart the service and check the log
```
sudo systemctl start nolusd && sudo journalctl -u nolusd -f --no-hostname -o cat
```
