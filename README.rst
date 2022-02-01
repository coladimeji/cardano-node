Cardano Nodes
-----------------------------


1. Install Cabal and GHC


# First, update packages and install Ubuntu dependencies.

sudo apt-get update -y
sudo apt-get -y install curl libsodium-dev build-essential pkg-config libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev zlib1g-dev make g++ tmux git jq wget libncursesw5 -y






# Install Cabal:

wget https://downloads.haskell.org/~cabal/cabal-install-3.2.0.0/cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz
tar -xf cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz
rm cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz cabal.sig
mkdir -p ~/.local/bin
mv cabal ~/.local/bin/





# Install GHC:

wget https://downloads.haskell.org/~ghc/8.6.5/ghc-8.6.5-x86_64-deb9-linux.tar.xz
tar -xf ghc-8.6.5-x86_64-deb9-linux.tar.xz
rm ghc-8.6.5-x86_64-deb9-linux.tar.xz
cd ghc-8.6.5
./configure
sudo make install





#Update PATH to include Cabal and GHC:

echo PATH="~/.local/bin:$PATH" >> ~/.bashrc
source ~/.bashrc


# Update cabal and verify the correct versions were installed successfully:
# At this moment Cabal library should be version 3.2.0.0 and GHC should be version 8.6.5

cabal update
cabal -V
ghc -V

---------------------------------------------------------------------
2. Build the node from source code


# Download source code and switch to the latest tag. In this case, use release/1.14.2

cd ~
git clone https://github.com/input-output-hk/cardano-node.git
cd cardano-node
git fetch --all --tags
git tag
git checkout tags/1.14.2


# Build the cardano-node from source code:


echo -e "package cardano-crypto-praos\n flags: -external-libsodium-vrf" > cabal.project.local
cabal install cardano-node cardano-cli  --overwrite-policy=always

# Building process may take a few minutes up to a few hours depending on your computer's processing power.



# Copy cardano-cli and cardano-node files into bin directory.

sudo cp $HOME/.cabal/bin/cardano-cli /usr/local/bin/cardano-cli
sudo cp $HOME/.cabal/bin/cardano-node /usr/local/bin/cardano-node

# version check:

cardano-node version
cardano-cli version

-----------------------------------------------------

3.1 Configure the block-producer node and the relay nodes


# Make relay folders and copy the json files

mkdir relaynode1
mkdir relaynode2
cp shelley_testnet-*.json relaynode1
cp shelley_testnet-*.json relaynode2


# Update relaynode1 (shelley_testnet-topology.json) with the following:

cat > relaynode1/shelley_testnet-topology.json << EOF 
 {
    "Producers": [
      {
        "addr": "127.0.0.1",
        "port": 8080,
        "valency": 2
      },
      {
        "addr": "127.0.0.1",
        "port": 8082,
        "valency": 2
      },
      {
        "addr": "relays-new.shelley-testnet.dev.cardano.org",
        "port": 3001,
        "valency": 2
      }
    ]
  }
EOF



# Update relaynode2

cat > relaynode2/shelley_testnet-topology.json << EOF 
 {
    "Producers": [
      {
        "addr": "127.0.0.1",
        "port": 8080,
        "valency": 2
      },
      {
        "addr": "127.0.0.1",
        "port": 8081,
        "valency": 2
      },
      {
        "addr": "relays-new.shelley-testnet.dev.cardano.org",
        "port": 3001,
        "valency": 2
      }
    ]
  }
EOF




# Update the block-producer node


cat > shelley_testnet-topology.json << EOF 
 {
    "Producers": [
      {
        "addr": "127.0.0.1",
        "port": 8081,
        "valency": 2
      },
      {
        "addr": "127.0.0.1",
        "port": 8082,
        "valency": 2
      }
    ]
  }
EOF




### reminder:
### Be sure your ports are corectly forwarded (8081 and 8082) and your firewall is configured


-----------------------------------------------------------------------------------------------

4. Create startup scripts


# block-producing node:

cat > startBlockProducingNode.sh << EOF 
DIRECTORY=~/cardano-my-node
PORT=8080
HOSTADDR=0.0.0.0
TOPOLOGY=\${DIRECTORY}/shelley_testnet-topology.json
DB_PATH=\${DIRECTORY}/db
SOCKET_PATH=\${DIRECTORY}/db/socket
CONFIG=\${DIRECTORY}/shelley_testnet-config.json
cardano-node run --topology \${TOPOLOGY} --database-path \${DB_PATH} --socket-path \${SOCKET_PATH} --host-addr \${HOSTADDR} --port \${PORT} --config \${CONFIG}
EOF



# relaynode1:

cat > relaynode1/startRelayNode1.sh << EOF 
DIRECTORY=~/cardano-my-node/relaynode1
PORT=8081
HOSTADDR=0.0.0.0
TOPOLOGY=\${DIRECTORY}/shelley_testnet-topology.json
DB_PATH=\${DIRECTORY}/db
SOCKET_PATH=\${DIRECTORY}/db/socket
CONFIG=\${DIRECTORY}/shelley_testnet-config.json
cardano-node run --topology \${TOPOLOGY} --database-path \${DB_PATH} --socket-path \${SOCKET_PATH} --host-addr \${HOSTADDR} --port \${PORT} --config \${CONFIG}
EOF





# relaynode2:

cat > relaynode2/startRelayNode2.sh << EOF 
DIRECTORY=~/cardano-my-node/relaynode2
PORT=8082
HOSTADDR=0.0.0.0
TOPOLOGY=\${DIRECTORY}/shelley_testnet-topology.json
DB_PATH=\${DIRECTORY}/db
SOCKET_PATH=\${DIRECTORY}/db/socket
CONFIG=\${DIRECTORY}/shelley_testnet-config.json
cardano-node run --topology \${TOPOLOGY} --database-path \${DB_PATH} --socket-path \${SOCKET_PATH} --host-addr \${HOSTADDR} --port \${PORT} --config \${CONFIG}
EOF


----------------------------------------------------------
5. Start the node


# Open 3 terminals and run each in a separate terminal:


cd ~/cardano-my-node
chmod +x startBlockProducingNode.sh
./startBlockProducingNode.sh



cd ~/cardano-my-node
chmod +x relaynode1/startRelayNode1.sh
./relaynode1/startRelayNode1.sh



cd ~/cardano-my-node
chmod +x relaynode2/startRelayNode2.sh
./relaynode2/startRelayNode2.sh



