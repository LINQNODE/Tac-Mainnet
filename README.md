

# TAC Chain

![0G Image](https://i.ibb.co/wNcjWZ0R/Tac-Chain.png)

--- 

## Community Links

- [Discord](https://discord.gg/NYhFQ3xMrc)
- [TAC Chain Twitter](https://x.com/TacBuild)
- [TAC Chain Website](https://tac.build/)
- [TAC Chain Telegram](https://t.me/tacbuild)
- [Blockchain Explorer](https://explorer.linqnode.com/tac)

---

## 💻 System Requirements

| Components  | Minimum Requirements |
|-------------|----------------------|
| CPU         | 8 Cores               |
| RAM         | 16+ GB                 |
| Storage     | 500+ GB SSD            |


### ✅ System Update
```bash
sudo apt update
sudo apt install curl git jq lz4 build-essential make gcc snapd chrony tmux unzip bc -y
sudo apt upgrade -y
```

### ✅ Go Installation

> **Note:** If you already have Go 1.19+ installed, skip this step
# Check current Go version
```bash
go version
```

If Go is missing or version is below 1.19, run the following commands:

```bash
rm -rf $HOME/go
sudo rm -rf /usr/local/go
cd $HOME
curl https://dl.google.com/go/go1.24.5.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
go version
```

### ✅ TAC Chain Binary Installation
```
go version
```

### ✅  Dosyaları çekip derleyelim

```bash
cd $HOME
rm -rf tacchain
git clone https://github.com/TacBuild/tacchain.git
cd tacchain
git checkout v1.0.1
make build
```

### ✅ Cosmovisor Installation

```bash
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0

```

### ✅ Create genesis binary directory
```bash
mkdir -p /root/.tacchaind/cosmovisor/upgrades/v1.0.1/bin
cp build/tacchaind /root/.tacchaind/cosmovisor/upgrades/v1.0.1/bin/
cp build/tacchaind $HOME/.tacchaind/cosmovisor/genesis/bin/

```

### ✅ Create system symlinks

```bash
sudo ln -s $HOME/.tacchaind/cosmovisor/genesis $HOME/.tacchaind/cosmovisor/current -f
sudo ln -s $HOME/.tacchaind/cosmovisor/current/bin/tacchaind /usr/local/bin/tacchaind -f
```

### ✅ Create genesis binary directory
```bash
sudo tee /etc/systemd/system/tacchaind.service > /dev/null << EOF
[Unit]
Description=tacchaind node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.tacchaind"
Environment="DAEMON_NAME=tacchaind"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.tacchaind/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```

```
sudo systemctl daemon-reload
sudo systemctl enable tacchaind.service
```


### ✅ Node Configuration
Initialize node (replace NODE_NAME with your desired name)
```bash
tacchaind init NODE_NAME --chain-id tacchain_239-1
```

### ✅ İSet port configuration (optional - default is 26xxx, here we use 59xxx)

```bash
echo 'export TAC_PORT="59"' >> $HOME/.bash_profile
source $HOME/.bash_profile
```


### ✅ Genesis & Addrbook 
Download genesis file
```bash
curl -Ls https://ss.tac.nodestake.org/genesis.json > $HOME/.tacchaind/config/genesis.json
```

Download addrbook file
```bash
curl -Ls https://ss.tac.nodestake.org/addrbook.json > $HOME/.tacchaind/config/addrbook.json
```


### ✅ Peers and Pruning Settings
Peers configuration (seeds left empty)
```bash
SEEDS=""
PEERS="d146b7727aa3a91404c23faa149c35896ebd82f1@141.95.97.21:60256,a460c869b5132849dc761d862a728350a8712fba@63.251.232.230:45110,6a16de4127bb0881e7fb7bb72e5083dbd6015f07@109.123.108.141:45110,10550a03e4f7fa487c78fbd07e0770e2b0f085c7@64.46.115.78:58960,b047e2dd6b068d8190b574264c93bc3e270dcf76@107.6.89.134:55130,509286977a3c644b6fc36b1a5e0f3927bb3f1266@84.32.186.70:60256,0327e180e47c30c47b6a69bdd862fd249c701212@23.106.238.97:60256,338eca43dc3d17a5ec7bf0d4ad38883452849569@63.251.232.121:55130,0efae9d157f0ef60ad7d25507d6939799f832e34@69.4.239.26:58960"

sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.tacchaind/config/config.toml
```

Pruning settings
```bash
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.tacchaind/config/app.toml
```

Timeout settings
```bash
sed -i 's/timeout_commit = "5s"/timeout_commit = "2s"/' $HOME/.tacchaind/config/config.toml
```



### ✅ Port Configuration
> **Optional:** If you want to change ports from default 26xxx to 59xxx

```bash
sed -i.bak -e "s%:1317%:${TAC_PORT}317%g;
s%:8080%:${TAC_PORT}080%g;
s%:9090%:${TAC_PORT}090%g;
s%:9091%:${TAC_PORT}091%g;
s%:8545%:${TAC_PORT}545%g;
s%:8546%:${TAC_PORT}546%g;
s%:6065%:${TAC_PORT}065%g" $HOME/.tacchaind/config/app.toml

sed -i.bak -e "s%:26658%:${TAC_PORT}658%g;
s%:26657%:${TAC_PORT}657%g;
s%:6060%:${TAC_PORT}060%g;
s%:26656%:${TAC_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${TAC_PORT}656\"%;
s%:26660%:${TAC_PORT}660%g" $HOME/.tacchaind/config/config.toml
```

### ✅ Snapshot Download
> **Recommended:** This significantly speeds up synchronization


Download snapshot for fast synchronization
```bash
SNAP_NAME=$(curl -s https://ss.tac.nodestake.org/ | egrep -o ">20.*\.tar.lz4" | tr -d ">")
curl -o - -L https://ss.tac.nodestake.org/${SNAP_NAME} | lz4 -c -d - | tar -x -C $HOME/.tacchaind
```

### ✅ Node Start

```bash
sudo systemctl start tacchaind.service
```
### Follow logs

```bash
sudo journalctl -u tacchaind.service -f --no-hostname -o cat
```


### ✅ Sync Check

Check status (add --node if using custom port)
```bash
tacchaind status --node http://localhost:59657
```

Check sync status
```bash
tacchaind status --node http://localhost:59657 2>&1 | jq .sync_info.catching_up
```
If returns "false", node is fully synced


### ✅ Wallet Creation
Create new wallet
 
```bash
tacchaind keys add wallet_name
```

List wallets

```bash
tacchaind keys list
```

Show wallet address

```bash
tacchaind keys show wallet_name -a
```

Show EVM address

```bash
echo "0x$(tacchaind debug addr $(tacchaind keys show wallet_name -a) | grep hex | awk '{print $3}')"
```


### ✅ Balance Check

```bash
tacchaind query bank balances $(tacchaind keys show wallet_name -a) --node http://localhost:59657
```

### ✅ Validator Setup
> **Important:** Send TAC tokens to your wallet first! Minimum: 1 TAC + gas fees (~0.15 TAC)

```
cd
```

```
tacchaind tendermint show-validator
```


### ✅ Create Validator Transaction File
```bash
echo "{\"pubkey\":{\"@type\":\"/cosmos.crypto.ed25519.PubKey\",\"key\":\"$(tacchaind tendermint show-validator | grep -Po '\"key\":\s*\"\K[^"]*')\"},
    \"amount\": \"miktar000000000000000000utac\",
    \"moniker\": \"test\",
    \"identity\": \"\",
    \"website\": \"\",
    \"security\": \"\",
    \"details\": \"CR\",
    \"commission-rate\": \"0.1\",
    \"commission-max-rate\": \"0.2\",
    \"commission-max-change-rate\": \"0.01\",
    \"min-self-delegation\": \"1\"
}" > validatortx.json
```


### ✅ Create Validator
```bash
tacchaind tx staking create-validator validatortx.json \
    --from wallet_name \
    --chain-id tacchain_239-1 \
    --node http://localhost:59657 \
    --gas auto --gas-adjustment 1.4 \
    --fees 130000000000000000utac -y
```

### ✅ Delegate
```bash
tacchaind tx staking delegate tacvaloper15xm09… 1000000utac --from wallet_name --chain-id tacchain_239-1 --gas auto --fees 500utac -y
```

### ✅ Validator Check

Search for validator in list

```bash
tacchaind query staking validators --node http://localhost:59657 | grep "YOUR_MONIKER"
```

Check validator status

```bash
tacchaind query staking validator $(tacchaind tendermint show-validator) --node http://localhost:59657
```

### ✅ Useful Commands

 Node status

```bash
sudo systemctl status tacchaind
```

Restart node
```bash
sudo systemctl restart tacchaind
```

View logs
```bash
sudo journalctl -u tacchaind -f --no-pager
```

Stop node
```bash
sudo systemctl stop tacchaind
```

Check disk usage
```bash
du -sh $HOME/.tacchaind/
```

## Contributing

If you encounter issues or have improvements, please:

1. Check the [troubleshooting section](#useful-commands)
2. Review the [official documentation](https://docs.tac.build)
3. Join the community channels for support


## License

This guide is provided as-is for educational purposes. Please refer to the official TAC documentation for the most up-to-date information.
