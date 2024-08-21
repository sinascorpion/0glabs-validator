![image](https://raw.githubusercontent.com/sinascorpion/0glabs-validator/main/0g.png)



### Official Links
* [OG Discord](https://discord.gg/0glabs)
* [Our telegram channel](https://t.me/dropper7)


## 🛠️System Requirements
| Ram | cpu     | disk                      |
| :-------- | :------- | :-------------------------------- |
| `8GB`      | `4Core` | `500+ SSD` |


## 📌Step 1: Installation packeges and dependencies
```shell
sudo apt update
sudo apt install -y lz4 jq make git gcc build-essential curl chrony unzip gzip snapd tmux bc

# Install Go if you need
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```
## 📌Step 2: Set moniker and install node
Give your validator a name by which you can find yourself in explorer, put it in ""
```shell
MONIKER="" 
```
After that, insert the following node installation command

```shell
# Clone project repository
cd $HOME
rm -rf 0g-chain
git clone https://github.com/0glabs/0g-chain.git
cd 0g-chain
git checkout v0.3.1
git submodule update --init
make install

# Set node CLI configuration
0gchaind config chain-id zgtendermint_16600-2
0gchaind config keyring-backend test
0gchaind config node tcp://localhost:26657

# Initialize the node
0gchaind init "$MONIKER" --chain-id zgtendermint_16600-2

# Download genesis and addrbook files
wget https://snapshots-testnet.unitynodes.com/0gchain-testnet/addrbook.json -O $HOME/.0gchain/config/addrbook.json
wget https://snapshots-testnet.unitynodes.com/0gchain-testnet/genesis.json -O $HOME/.0gchain/config/genesis.json

# Set seeds, peers
PEERS="e371f26305869fd8294f6e57dc01ffbbd394a5ac@156.67.80.182:26656,f8e73164ef67ec5288f663b271d320f303832b49@149.102.147.164:12656,c45a79a6e28fbee2b35b55bc2e18644fe4d20bb8@62.171.131.80:12656,7baa9325f18259079d701d649d22221232dd7a8d@116.202.51.84:26656,cd1d5fc0f6f35d0ef7d640c33b5159d84d07bd5c@161.97.110.100:12656,dbb44850914d0507e082ea81efd32662f883b222@62.169.26.33:26656,3be5290378f4ef5a5793bde6f5b7cf198f215366@65.108.200.101:26656,908a7a4f23d8a0933dbf11cbb0dbfe36e16f7d03@185.209.228.241:26646,c0cfc7c9d0cab4562e1933adf9fcc62f659f1b78@94.16.105.248:13456,a9d070c0c5900c3734a57c985f06098088b46583@213.199.32.62:12656,ecd31d198e658512967d964d8b80c1c8cc29a1d4@5.189.182.240:12656,0a827d0e1966731fd8680490601f49e5e9dc7130@158.220.109.21:26656,b517215f5542d9978981d63b7b926f8d70d9c9db@62.171.167.145:12656,276186e07dd59c28306286156ce8738d357e761a@109.199.100.144:12656,a4cc54c65e2f14e1ac28103c816d6e4f8f4f06e0@65.108.6.59:26656"
SEEDS="81987895a11f6689ada254c6b57932ab7ed909b6@54.241.167.190:26656,010fb4de28667725a4fef26cdc7f9452cc34b16d@54.176.175.48:26656,e9b4bc203197b62cc7e6a80a64742e752f4210d5@54.193.250.204:26656,68b9145889e7576b652ca68d985826abd46ad660@18.166.164.232:26656"
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.0gchain/config/config.toml

# Set minimum gas price
sed -i "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0ua0gi\"/" $HOME/.0gchain/config/app.toml

# Set pruning
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.0gchain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.0gchain/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.0gchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.0gchain/config/app.toml

# Download latest chain data snapshot
curl https://snapshots-testnet.unitynodes.com/0gchain-testnet/0gchain-testnet-latest.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.0gchain

# Create service
sudo tee /etc/systemd/system/0gchaind.service > /dev/null << EOF
[Unit]
Description=0G node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which 0gchaind) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

### Start service and run node
echo ""
printColor blue "[6/6] Start service and run node"

sudo systemctl daemon-reload
sudo systemctl enable 0gchaind.service
sudo systemctl start 0gchaind.service
```

## 📌Step 3: Node Health Check
Follow the commands to check if your node is working properly

1. Check version
```shell
0gchaind version
```

2. View sync status
```shell
local_height=$(0gchaind status | jq -r .sync_info.latest_block_height); network_height=$(curl -s https://rpc.0gchain-testnet.unitynodes.com/status | jq -r .result.sync_info.latest_block_height); blocks_left=$((network_height - local_height)); echo "Your node height: $local_height"; echo "Network height: $network_height"; echo "Blocks left: $blocks_left"
```
Blocks left - 0-1 everything is fine and your node catches up with the last block of the network.

3. Check logs
```shell
tail -f -n 100 $HOME/.0gchain/log/chain.log
```
If your node is installed and fully synchronized with the network, proceed with the creation of the validator.


### 📝 Create Validator

## 📌Step 1: Create or import wallet

```shell
0gchaind keys add wallet --eth
```
!Save all information after entering the command, without this you will not be able to restore data to the wallet. SAVE SEED PHRASE (12 words).

Or if you had it before you can recover
* Replace WALLET_NAME
```shell
0gchaind keys add WALLET_NAME --eth --recover
```
## 📌Step 2: Request test tokens to your wallet address
We request the private key from our EVM address
```shell
0gchaind keys unsafe-export-eth-key wallet # or your wallet name
```

Press Y and enter
We keep this private key with us
Go to the metamask and import the wallet using it:
We open the [Faucet 0G Link 🚰](https://faucet.0g.ai/) and request tokens to the metamask address we received

## 📌Step 3: Check your balance in terminal
```shell
0gchaind q bank balances $(0gchaind keys show $WALLET_NAME -a)
```
## 📌Step 4: Create validator
Replace YOUR_WALLET with your wallet name
```shell
0gchaind tx staking create-validator \
--amount 1000000ua0gi \
--chain-id=zgtendermint_16600-2 \
--pubkey $(0gchaind tendermint show-validator) \
--moniker "$NODE_MONIKER" \
--identity "" \
--website "" \
--details "" \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from YOUR_WALLET \
--gas-prices=0.25ua0gi  \
--gas-adjustment=1.5 \
--gas=auto \
-y
```

## 📌Step 5: Backup. 
```shell
cat $HOME/.0gchain/config/priv_validator_key.json
```

### Delegate tokens

```shell
0gchaind tx staking delegate $(0gchaind keys show WALLET_NAME --bech val -a) 1000000ua0gi --from WALLET_NAME -y
```

--------------------------------------
### Useful commands

* Replace WALLET_NAME

## Unjail Node (if your node is jailed)

```shell
0gchaind tx slashing unjail --from WALLET_NAME --chain-id zgtendermint_16600-2 --gas-adjustment 1.5 --gas auto --gas-prices 0.00252ua0gi -y
```

## Active Validators list

```shell
0gchaind q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```

## inActive Validators list

```shell
0gchaind q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```

## Wallet Balance

```shell
0gchaind q bank balances $(0gchaind keys show WALLET_NAME -a)
```

## Your Validator Status

```shell
0gchaind q staking validator $(0gchaind keys show WALLET_NAME --bech val -a)
```


## Delete Node

```shell
sudo systemctl stop 0gchaind.service
sudo systemctl disable 0gchaind.service
sudo rm /etc/systemd/system/0gchaind.service
rm -rf $HOME/.0gchain $HOME/0g-chain
```

## Check the validators list in Explorer
https://explorer.coinhunterstr.com/0G-Newton/staking


### fill out the google form:
https://docs.google.com/forms/d/e/1FAIpQLScsa1lpn43F7XAydVlKK_ItLGOkuz2fBmQaZjecDn76kysQsw/viewform?ts=6617a343
