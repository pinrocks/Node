## State Sync
```
sudo systemctl stop echelond

cp $HOME/.echelond/data/priv_validator_state.json $HOME/.echelond/priv_validator_state.json.backup
echelond tendermint unsafe-reset-all --home $HOME/.echelond --keep-addr-book

SNAP_RPC="https://mainnet-echelon-rpc.konsortech.xyz:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

peers="333f8e058a9294555b531b8ed66c2c7703f226a1@mainnet-echelon.konsortech.xyz:24656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.echelond/config/config.toml

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.echelond/config/config.toml

mv $HOME/.echelond/priv_validator_state.json.backup $HOME/.echelond/data/priv_validator_state.json

sudo systemctl restart echelond
sudo journalctl -u echelond -f --no-hostname -o cat
```
