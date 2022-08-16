### Setup a [flashbots/mev-boost](https://github.com/flashbots/mev-boost#usage) systemd service (Goerli testnet)

1. Install go 1.18+
```zsh 
# using gobrew
curl -sLk https://git.io/gobrew | sh -

# add gobrew to path
echo 'export PATH="$HOME/.gobrew/current/bin:$HOME/.gobrew/bin:$PATH"' >> ~/.bashrc

# add go packages to path
echo 'export PATH=${PATH}:`go env GOPATH`/bin' >> ~/.bashrc

# set go version
gobrew use 1.18@latest

# reload .bashrc
. ~/.bashrc
```
2. Install flashbots/mev-boost
```zsh
go install github.com/flashbots/mev-boost@latest

# sym link mev-boost to /usr/local/bin
cd /usr/local/bin && sudo ln -s ~/go/bin/mev-boost .
```
3. Create mevb user
```zsh
sudo useradd --no-create-home --shell /bin/false mevb
```
4. Create **mevb-goerli.service** systemd service
```zsh
sudo tee /etc/systemd/system/mevb-goerli.service > /dev/null <<EOF
[Unit]
Description=MEV-Boost Goerli Test
After=network.target
Wants=network.target

[Service]
User=mevb
Group=mevb
Type=simple
Restart=always
RestartSec=5
ExecStart=mev-boost -goerli -relay-check -relays https://0xafa4c6985aa049fb79dd37010438cfebeb0f2bd42b115b89dd678dab0670c1de38da0c4e9138c9290a398ecd9a0b3110@builder-relay-goerli.flashbots.net

[Install]
WantedBy=default.target
EOF
```
5. Start and enable service
```zsh
sudo systemctl start mevb-goerli.service
sudo systemctl enable mevb-goerli.service
```
6. Check the logs
```zsh
journalctl -fu mevb-goerli.service
```
