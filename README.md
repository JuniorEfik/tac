# How To Join The TAC Network As a Node Operator
For this guide, we'll be focusing on Saint Petersburg Testnet since the old one will soon be phased out.

### Network Details
- Chain ID: 2391
- EVM JSON RPC: https://spb.rpc.tac.build
- RPC: https://spb.tendermint.rpc.tac.build
- EVM Explorer: https://spb.explorer.tac.build/
- Cosmos Explorer: https://bd-explorer.tac-spb.ankr.com/
- Faucet: https://spb.faucet.tac.build/
- Staking UI: https://public1.rollup.asphere.xyz/

### Hardware Requirements
- CPU: 8 cores
- RAM: 16GB (for RPC nodes) / 32GB (for validator nodes)
- Storage: 500GB NVMe SSD

### Install Dependencies
Update packages:
```
sudo apt-get update && sudo apt-get upgrade -y
```
Install go:
```
sudo rm -rf /usr/local/go
wget https://go.dev/dl/go1.23.6.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.23.6.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
rm go1.23.6.linux-amd64.tar.gz
```
Install packages:
```
sudo apt install -y jq curl screen git
```
Verify installations:
```
go version  
jq --version
curl --version
screen --version
git --version
```
Install Tacchain:
```
git clone https://github.com/TacBuild/tacchain.git
cd tacchain
git checkout v0.0.8
make install
```
Initalize network folder (replace YOUR_NODE_NAME with a unique name of your choice):
```
tacchaind init <YOUR_NODE_NAME> --chain-id tacchain_2391-1 --home .testnet
```
Modify config.toml:
```
nano .testnet/config/config.toml
```
Search and replace the lines of code with the following:
```
timeout_commit = "2s"

persistent_peers = "9c32b3b959a2427bd2aa064f8c9a8efebdad4c23@206.217.210.164:45130,04a2152eed9f73dc44779387a870ea6480c41fe7@206.217.210.164:45140,5aaaf8140262d7416ac53abe4e0bd13b0f582168@23.92.177.41:45110,ddb3e8b8f4d051e914686302dafc2a73adf9b0d2@23.92.177.41:45120"
```
Hit Ctrl+X; then Y; then press Enter to save
Go back to main directory:
```
cd ../../
```
Fetch genesis:
```
curl https://raw.githubusercontent.com/TacBuild/tacchain/refs/heads/main/networks/tacchain_2391-1/genesis.json > .testnet/config/genesis.json
```
Create a screen session:
```
screen -S tacchain
```
Start node:
```
tacchaind start --chain-id tacchain_2391-1 --home .testnet
```
Minimize screen by hitting Ctrl+A+D. Your screen can be viewed using:
```
screen -r tacchain
```
Once your node is fully synced, you can register as a validator. You should get some faucets from [Faucet](https://spb.faucet.tac.build/).
Import your private key:
```
tacchaind --home .testnet keys unsafe-import-eth-key validator <PRIVATE_KEY> --keyring-backend test
```
Create validator transaction:
```
echo "{\"pubkey\":$(tacchaind --home .testnet tendermint show-validator),\"amount\":\"1000000000000000000utac\",\"moniker\":\"<YOUR_NODE_NAME>\",\"identity\":null,\"website\":null,\"security\":null,\"details\":null,\"commission-rate\":\"0.1\",\"commission-max-rate\":\"0.2\",\"commission-max-change-rate\":\"0.01\",\"min-self-delegation\":\"1\"}" > validatortx.json

tacchaind --home .testnet tx staking create-validator validatortx.json --from validator --keyring-backend test -y
```
Delegate more tokens (optional):
```
tacchaind --home .testnet tx staking delegate $(tacchaind --home .testnet q staking validators --output json | jq -r '.validators[] | select(.description.moniker == "<YOUR_NODE_NAME>") | .operator_address') 1000000000000000000utac --keyring-backend test --from validator -y
```
Participating in governance (optional):
```
# list all proposals on chain
tacchaind q gov proposals

# once you have identified the proposal (need `proposal_id`) you can place your vote
# In the following example we vote with 'yes', alternatively you can vote with 'no'
tacchaind tx gov vote <PROPOSAL_ID> yes --from validator
```
For improved security, validators should implement a sentry node architecture (which I'll be skipping)

### Errors
- ERR failure when running app err="error during handshake: error on replay: UPGRADE "v0.0.9" NEEDED at height: 872601: add eth_getBlockReceipts, disable x/feemarket, remove wasmd". Fix with:
```
git checkout v0.0.9
make install
tacchaind start --chain-id tacchain_2391-1 --home .testnet
```
-  ERR error in proxyAppConn.FinalizeBlock err="UPGRADE \"v0.0.10\" NEEDED at height: 939826: enable x/feemarket tx fee checker" module=state server=node
panic: Failed to process committed block (939826:81A87D8AD53ED5F599866B1F382D6A853D9063589A2048669F98C5D5C896D0C8): UPGRADE "v0.0.10" NEEDED at height: 939826: enable x/feemarket tx fee checker. Fix with:
```
git checkout v0.0.10
make install
tacchaind start --chain-id tacchain_2391-1 --home .testnet
```
