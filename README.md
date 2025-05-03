# aztec-sequencer-setup-guide

- Pc or VPS
- docker 
- at least 5GB of ram.

## Node Guide :

This will work on Mac and Linux based devices.

### Step 1 : Install dependencies 

MacOS

- Install Homebrew if needed
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
- Install Docker & Socat
```bash
brew install socat
brew install --cask docker
```

Linux OS
```bash
sudo apt update -y && sudo apt upgrade -y
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y && sudo apt upgrade -y

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
- Test Docker
```bash
sudo docker run hello-world
```
```bash
sudo systemctl enable docker
sudo systemctl restart docker
```
```bash
sudo apt update
sudo apt install -y curl git socat docker.io docker-compose
sudo systemctl enable --now docker
```

### Step 2 : Install Aztec Cli

```bash
bash -i <(curl -s https://install.aztec.network)
```

MacOS

```bash
 echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```
or for Bash

```bash
echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
```

Linux OS

```bash
echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
```
Test the cli functionality 

```bash
aztec --version
```
```bash
aztec-up alpha-testnet
```

Output should be in this format 0.8.5 or so.

### Step 3 : Get your public IP and expose your ports

```bash
curl ifconfig.me
```

Save the Ip gotten.

Open your ports 
```bash
# Firewall
ufw allow 22
ufw allow ssh
ufw enable
```
```bash
# Sequencer
ufw allow 40400
ufw allow 8080
```

### Step 4 : create and env file to store sensitive details

Get your own Sepolia RPC URL from https://www.alchemy.com/ & Sepolia BEACON URL from https://rpc.drpc.org/eth/sepolia/beacon

```bash
nano ~/.aztec-sequencer.env
```

Create new wallet and Paste this into the file and edit with your private RPCs and beacon or use public.
You can use public beacon rpc (lodestar-sepolia.chainsafe.io)

```bash
# Your Infura Execution RPC (sepolia)
ETHEREUM_HOSTS=sepl rpc paste here

# Reliable Public Beacon Chain (Consensus) Endpoint
L1_CONSENSUS_HOST_URLS=lodestar-sepolia.chainsafe.io

# Replace with your values
P2P_IP=your ip address
SEQUENCER_VALIDATOR_PRIVATE_KEY=YourPrivateKey
SEQUENCER_COINBASE=eth wallet
```
Save the file with control x + y + enter.

Load the .env file 

```bash
source ~/.aztec-sequencer.env
```

### Step 5 : Start The Sequencer Node

```bash
screen -S Aztec
```

```bash
aztec start --node --archiver --sequencer \
  --network alpha-testnet \
  --l1-rpc-urls "sepl rpc" \
  --l1-consensus-host-urls "beacon rpc" \
  --sequencer.validatorPrivateKey "private key" \
  --sequencer.coinbase "wallet address" \
  --p2p.p2pIp "your ip" \
  --p2p.maxTxPoolSize 1000000000
```
![Image 5-1-25 at 8 56 PM](https://github.com/user-attachments/assets/fb9372d1-4446-4c04-b2f8-5406a0ac0c09)

### Step 6 : Register as an apprentice on discord

Wait 10-15mins for your sequencer node to come up then proceed with this step.

Go to the Aztec discord operators channel :

(https://discord.com/invite/aztec)

### Step A: Get the latest proven block number:

```bash
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getL2Tips","params":[],"id":67}' \
http://localhost:8080 | jq -r ".result.proven.number"
```

- Save this block number for the next steps
- Example of the output: 20834

### Step B: Generate your sync proof

```bash
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getArchiveSiblingPath","params":["BLOCK_NUMBER","BLOCK_NUMBER"],"id":67}' \
http://localhost:8080 | jq -r ".result"
```

Replace both BLOCK_NUMBER with your number

### Step C: Register with Discord

Type the following command in this Discord server: /operator start
After typing the command
address: Your validator address (Ethereum Address)
block-number: Block number for verification (Block number from Step 1)
proof: Your sync proof (step 2 result)

Then you'll get your Apprentice Role.

### Step 7 : Register as a validator 

```bash
aztec add-l1-validator \
  --l1-rpc-urls "$ETHEREUM_HOSTS" \
  --private-key "$SEQUENCER_VALIDATOR_PRIVATE_KEY" \
  --attester "$SEQUENCER_COINBASE" \
  --proposer-eoa "$SEQUENCER_COINBASE" \
  --staking-asset-handler 0xF739D03e98e23A7B65940848aBA8921fF3bAc4b2 \
  --l1-chain-id 1 

![439840617-ed683c0c-251e-4049-9479-fd4ec86ee640](https://github.com/user-attachments/assets/935f10a4-52bc-4bcf-be0a-a08331e2aa86)


