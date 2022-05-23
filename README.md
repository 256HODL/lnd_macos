# Build a Ligthing Node on macOS with LND and a Full Node

## Step 1: Build Bitcoin Core from source with ZMQ notifications enabled.

**Bitcoin Core macOS Build Guide**
- https://github.com/bitcoin/bitcoin/blob/master/doc/build-osx.md

Summary of commands to build a full node:

```
## Does not include a GUI or wallet. See the Bitcoin Core Build Guide if you'd like to enable this functionality.

# Install Xcode
xcode-select --install

# Install Brew (skip if you already have this installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Dependencies
brew install automake libtool boost pkg-config libevent

# Clone Bitcoin repository
git clone https://github.com/bitcoin/bitcoin.git && cd "$_"

# Install ZMQ Dependencies
brew install zeromq

# Install Python (skip if you already have Python3 installed)
brew install python

# Build Bitcoin Core with no Wallet or GUI
./autogen.sh
./configure --without-wallet --with-gui=no

# Compile
make
make check

# Create empty configuration file
mkdir -p "/Users/${USER}/Library/Application Support/Bitcoin"
touch "/Users/${USER}/Library/Application Support/Bitcoin/bitcoin.conf"
chmod 600 "/Users/${USER}/Library/Application Support/Bitcoin/bitcoin.conf"

# Run
./src/bitcoind
```

## Step 2: Build LND from source
```
# Install Go
brew install go

# Append this to your .zshrc or .bashrc file in your home directory.
export GOPATH=~/gocode
export PATH=$PATH:$GOPATH/bin
export PATH=$PATH:~/go/bin

# Install LND
git clone https://github.com/lightningnetwork/lnd && cd "$_"
make install tags="autopilotrpc chainrpc invoicesrpc routerrpc signrpc walletrpc watchtowerrpc wtclientrpc"
```

## Step 3: Setup RPC credentials
Ensure you are in the `/bitcoin` directory in your Git repo.

```
cd /bitcoin
python3 share/rpcauth/rpcauth.py btcrpcuser
```

This will output some info. Take note of the auth string after `btcrpcuser:` as this will be used in the bitcoin.conf and lnd.conf file.

```
rpcauth=btcrpcuser:SOME_AUTH_STRING
Your password:
YOUR_RPC_PASSWORD
```

## Step 4: Configure your bitcoin.conf file
File is located in:

`/Users/${USER}/Library/Application Support/Bitcoin/bitcoin.conf`

Use a text editor or vi/vim/nano to include the below, replacing `SOME_AUTH_STRING` with the string returned in Step 3.

```
rpcuser=btcrpcuser
rpcpassword=SOME_AUTH_STRING
zmqpubrawblock=tcp://127.0.0.1:28332
zmqpubrawtx=tcp://127.0.0.1:28333
txindex=1
```

## Step 5: Configure your lnd.conf file
Create the LND directory and config file
```
mkdir ~/.lnd/ && touch ~/.lnd/lnd.conf
```

Use a text editor or vi/vim/nano to include the below, replacing `SOME_AUTH_STRING` with the string returned in Step 3.

```
[Bitcoin]
rpcuser=btcrpcuser
rpcpassword=SOME_AUTH_STRING
bitcoin.node=bitcoind
bitcoin.active=true
bitcoin.mainnet=true
listen=0.0.0.0:9735
listen=[::1]:9736
```

## Step 6: Run LND
Run the following command via CLI:
```
lnd
```
As we started LND for the first time, we will have to create a wallet with the command:

```
lncli create
```

Follow the steps carefully and write down your 24 word mnemonic seed phrase. Store it in a secure place. You may also choose a password, which will be used to encrypt this wallet on your disk. 

Later, we will need to unlock this wallet everytime we restart lnd using the command 

```
lncli unlock
```

## Step 7: Deposit some Bitcoin
Run the following command via CLI:
```
lncli newaddress p2wkh
```
This will output a main-chain wallet address which you can sends sats to.

Any sats sent to these addresses will now show up in the commmand:

```
lncli walletbalance
```

## Step 8: Find a peer
Check https://1ml.com/ for some open public peers. Note the public key and the IP address and port.

For example:
- https://1ml.com/node/03a6680e79e30f050d4f32f1fb9d046cc6efb5ed4cc99eeedba6b2e89cbf838691

`PUBLIC_NODE_PUBLIC_KEY:` 03a6680e79e30f050d4f32f1fb9d046cc6efb5ed4cc99eeedba6b2e89cbf838691

`IP_ADDRESS:PORT:` 85.216.202.47:9735

```
lncli openchannel --conf_target 6 --node_key PUBLIC_NODE_PUBLIC_KEY --connect IP_ADDRESS:PORT --local_amt AMOUNT_OF_SATS
```

## References
- Bitcoin Core Github
https://github.com/bitcoin/bitcoin

- LND Documentation
https://docs.lightning.engineering/
