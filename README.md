[Setup](#Setup) - [Updating mev-boost](#Updating-mev-boost) - [Uninstall](#Uninstall ) - [CL Client Config Resources](#Consensus-Layer-Client-Config-Resources)

### Note
- This does not cover Consensus Client configurations necessary to use mev-boost as a builder

### Setup
#### [flashbots/mev-boost](https://github.com/flashbots/mev-boost#usage) systemd service (Mainnet)

1. Install go 1.18+
```zsh 
# using gobrew
curl -sLk https://git.io/gobrew | sh -

# add gobrew to path
echo 'export PATH="$HOME/.gobrew/current/bin:$HOME/.gobrew/bin:$PATH"' >> $HOME/.bashrc

# add go packages to path
echo 'export PATH=${PATH}:`go env GOPATH`/bin' >> $HOME/.bashrc

# set/install go version
gobrew use 1.18@latest

# reload .bashrc
. $HOME/.bashrc
```
2. Install flashbots/mev-boost
```zsh
go install github.com/flashbots/mev-boost@latest

# sym link mev-boost to /usr/local/bin
cd /usr/local/bin && sudo ln -s $HOME/go/bin/mev-boost . && cd $HOME
```
3. Create mevb user
```zsh
sudo useradd --no-create-home --shell /bin/false mevb
```
4. Create mev-boost relay config directory
```zsh
sudo mkdir /etc/mev-boost
```
5. Create relay-config.env environment variables
```zsh
sudo tee /etc/mev-boost/relay-config.env > /dev/null <<EOF
BLOXROUTE_ETHICAL="https://0xad0a8bb54565c2211cee576363f3a347089d2f07cf72679d16911d740262694cadb62d7fd7483f27afd714ca0f1b9118@bloxroute.ethical.blxrbdn.com"
FLASHBOTS="https://0xac6e77dfe25ecd6110b8e780608cce0dab71fdd5ebea22a16c0205200f2f8e2e3ad3b71d3499c54ad14d6c21b41a37ae@boost-relay.flashbots.net"
EOF
```
6. Set recursive ownership of /etc/mev-boost to mevb user
```zsh
sudo chown -R mevb:mevb /etc/mev-boost/
```
7. Create **mevb.service** systemd service
```zsh
sudo tee /etc/systemd/system/mevb.service > /dev/null <<EOF
# /etc/systemd/system/mevb.service 

[Unit]
Description=MEV-Boost Mainnet
After=network.target
Wants=network.target

[Service]
User=mevb
Group=mevb
Type=simple
Restart=always
RestartSec=5
EnvironmentFile=/etc/mev-boost/relay-config.env
ExecStart=mev-boost -mainnet -relay-check -relays \${BLOXROUTE_ETHICAL},\${FLASHBOTS}

[Install]
WantedBy=default.target
EOF
```
8. Start and enable service
```zsh
sudo systemctl start mevb.service
sudo systemctl enable mevb.service
```
9. Check the logs
```zsh
journalctl -fu mevb.service

# You should see something like:

# <timestamp> <youruser> systemd[1]: Started MEV-Boost Mainnet.
# <timestamp> <youruser> mev-boost[21499]: time="<timestamp>" level=info msg="mev-boost v0.8.2" module=cli
# <timestamp> <youruser> mev-boost[21499]: time="<timestamp>" level=info msg="Using genesis fork version: 0x00000000" module=cli
# <timestamp> <youruser> mev-boost[21499]: time="<timestamp>" level=info msg="using 2 relays" module=cli relays="[{0xad0a8bb54565c2211cee576363f3a347089d2f07cf72679d16911d740262694cadb62d7fd7483f27afd714ca0f1b9118 https://0xad0a8bb54565c2211cee576363f3a347089d2f07cf72679d16911d740262694cadb62d7fd7483f27afd714ca0f1b9118@bloxroute.ethical.blxrbdn.com} {0xac6e77dfe25ecd6110b8e780608cce0dab71fdd5ebea22a16c0205200f2f8e2e3ad3b71d3499c54ad14d6c21b41a37ae https://0xac6e77dfe25ecd6110b8e780608cce0dab71fdd5ebea22a16c0205200f2f8e2e3ad3b71d3499c54ad14d6c21b41a37ae@boost-relay.flashbots.net}]"
# <timestamp> <youruser> mev-boost[21499]: time="<timestamp>" level=info msg="Checking relay" module=service relay="https://0xad0a8bb54565c2211cee576363f3a347089d2f07cf72679d16911d740262694cadb62d7fd7483f27afd714ca0f1b9118@bloxroute.ethical.blxrbdn.com"
# <timestamp> <youruser> mev-boost[21499]: time="<timestamp>" level=info msg="Checking relay" module=service relay="https://0xac6e77dfe25ecd6110b8e780608cce0dab71fdd5ebea22a16c0205200f2f8e2e3ad3b71d3499c54ad14d6c21b41a37ae@boost-relay.flashbots.net"
# <timestamp> <youruser> mev-boost[21499]: time="<timestamp>" level=info msg="listening on localhost:18550" module=cli
```

### Updating mev-boost

1. Update mev-boost binary
```zsh
go install github.com/flashbots/mev-boost@latest
```
2. Check your version
```zsh
mev-boost --version
# mev-boost v0.8.2
```
2. Restart mevb.service
```zsh
sudo systemctl reload-or-restart mevb.service
```


### Uninstall 

- Note: if you are using mev-boost as your builder for your consensus client, make sure to update your configs for those clients before removing mev-boost

1. Disable and stop mevb.service
```zsh
sudo systemctl disable mevb.service
sudo systemctl stop mevb.service
```
2. Remove /etc/mev-boost directory
```zsh
sudo rm -rf /etc/mev-boost
```
3. Remove /etc/systemd/system/mevb.service unit file
```zsh
sudo rm -r /etc/systemd/system/mevb.service
```
4. Remove mevb user
```zsh
sudo userdel mevb
```
5. (Optional) Remove gobrew and $HOME/go dir
```zsh
rm -rf $HOME/.gobrew
rm -rf $HOME/go
```
6. Remove mev-boost symlink from /usr/local/bin
```zsh
sudo rm /usr/local/bin/mev-boost
```
7. Remove gobrew entries from $HOME/.bashrc
```zsh
vim $HOME/.bashrc
# remove these lines from the bottom of .bashrc
# export PATH="$HOME/.gobrew/current/bin:$HOME/.gobrew/bin:$PATH"
# export PATH=${PATH}:`go env GOPATH`/bin

# source your .bashrc
. $HOME/.bashrc
```

### Consensus Layer Client Config Resources

- [lighthouse](https://lighthouse-book.sigmaprime.io/builders.html)
- [teku](https://docs.teku.consensys.net/en/latest/HowTo/Builder-Network/)
- [prysm](https://docs.prylabs.network/docs/prysm-usage/parameters#validator-configuration)
- [nimbus](https://nimbus.guide/external-block-builder.html)
