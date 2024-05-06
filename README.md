**Disclaimer:** to anyone with essentially any degree of real coding and/or development experience it will be clear that I myself have next to none. This document is a revision of another guide that I followed to initially set up my node, with some extra notes that may be useful to others like me, who are not ultra familiar with Linux, CLI (command-line interface) operations, and cryptography.

# Setting Up a Quilibrium Ceremony Client Node

To set up a Quilibrium Ceremony Client node and start acquiring $QUIL, follow these simplified steps. This guide includes session management with `tmux` and comprehensive process management instructions, now ordered to minimize the need for restarts.

> *For those who may not be familiar, `tmux` is a way to essentially have multiple windows, or control multiple processes, within a CLI (command-line interface).* 

## 1. Provision a VPS (virtual private server)

> *For the Dawn phase, a server must have a minimum of 16GB of RAM, preferably 32 GB, 250GB of storage, preferably via SSD, and 50MBps symmetric bandwidth. For Intel/AMD, the baseline processor is a Skylake processor @ 3.4GHz with 12 dedicated cores. For ARM, the M1 line of Apple is a good reference. With Dusk, these minimum requirements will reduce significantly. As of 5/5/24 version 1.5.0 or "Dusk" has been delayed several times over the last few weeks, but appears that it will be updated very soon.*

- Choose a VPS provider and set up a server running **Ubuntu 22.04**.
  - I am currently running a [Perfect Quality Hosting]( https://pq.hosting/en/vps-vds-netherlands-meppel
  ) server. Scroll down and select the **Adamantium-NL** option. I recommend paying with crypto - *this is the way*.

> *Each service is a little different, but when setting up your VPS you'll be assigned an IP address and set a password to access your server. Some services have more options than others, but keep it as simple as possible.

- Once your VPS has been provisioned, connect via SSH from your computer's terminal by replacing `VPS_IP_ADDRESS` with that of your VPS in the following command:
  ```bash
  ssh root@VPS_IP_ADDRESS
  ```

## 2. Install Prerequisite Software

Execute the following commands to install necessary software:

  ```bash
  sudo apt -q update
  sudo apt-get install jq tmux -y  # Includes tmux installation
  sudo apt install git -y
  git --version  # Ensure git version is 2.34.1 or later
  ```

## 3. Install Golang

Download and install Golang with these commands:

  ```bash
  wget https://go.dev/dl/go1.20.14.linux-amd64.tar.gz
  sudo tar -xvf go1.20.14.linux-amd64.tar.gz -C /usr/local
  echo 'export GOROOT=/usr/local/go' >> ~/.bashrc
  echo 'export GOPATH=$HOME/go' >> ~/.bashrc
  echo 'export PATH=$GOPATH/bin:$GOROOT/bin:$PATH' >> ~/.bashrc
  source ~/.bashrc
  go version  # Verify installation with go version go1.20.14 linux/amd64
  ```

## 4. Clone the Ceremony Client Repo

Clone the repository and navigate to the node directory:

```bash
git clone https://github.com/QuilibriumNetwork/ceremonyclient.git
cd ceremonyclient/node
```

## 5. Configure the firewall to allow necessary traffic:

  ```bash
  sudo ufw allow 22 && sudo ufw allow 8336 && sudo ufw allow 8337 && sudo ufw allow 8338 && sudo ufw allow 8317 && sudo ufw allow 8316 && sudo ufw enable
  sudo ufw status  # Confirm firewall settings
  ```

## 6. Increase buffer size for better network performance:

  ```bash
  echo -e "\n# Increase buffer sizes for better network performance\nnet.core.rmem_max=600000000\nnet.core.wmem_max=600000000" | sudo tee -a /etc/sysctl.conf > /dev/null && sudo sysctl -p
  ```
Apply changes:

  ```bash
  sudo sysctl -p
  ```

## 7. Start Node in a tmux session

Initiate a tmux session:

```bash
tmux new -s quilnode
```

Start the node within the tmux session:

```bash
cd ~/ceremonyclient/node
GOEXPERIMENT=arenas go run ./...
```

### You are now running a quilibrium node!

## *** some notes on tmux ***

- Detach from the tmux session with `Ctrl+b` then `d` 
- Reattach to tmux session with `tmux a`
- Open a new window within the session with `Ctrl + b` then `c`
- Cycle through open windows with `Ctrl + b` then `p`
- Split window horizontally with `Ctrl + b` then `"`
- Split window vertically with `Ctrl + b` then `%`

# Node Management

## Restarting the node
- ### Auto Restart:

  - Navigate to the node window in tmux
  - Pause the node process with `Ctrl+z`
  - Make the auto script executable:

    ```bash
    chmod +x poor_mans_cd.sh
    ```
  - Start the auto script
    ```bash
    ./poor_mans_cd.sh
    ```
  - `poor_mans_cd.sh` should auto restart if your process goes down as well as auto update new github releases. Although in my exerience it sometimes doesn't work as planned and stalls out. If this happens, go for the manual restart.
- ### Manual Restart:

  - Navigate to the node window in tmux
  - Pause the node process with `Ctrl+z`
  - Find the PID of the node process. Look for the number for any proces ending in `/b001/exe/node `
    ```bash
    ps -aux | grep go # PID number for the process ending in /b001/exe/node
    ```
  - Use the PID to kill the process:
    ```bash
    kill PID
    ```
  - Recheck the node processes and confirm deletion. If not deleted force kill with:
    ```bash
    kill -9 PID
    ```
  - Now restart the node with:

    ```bash
    GOEXPERIMENT=arenas go run ./...
    ```

## Configure gRPC to enable node query:

- Edit `.config/config.yml`:

  ```bash
  nano ~/ceremonyclient/node/.config/config.yml
  ```
- Scroll to the bottom with the arrow keys to the line, `listenGrpcMultiaddr: ""`, and replace with:
  ```bash
  listenGrpcMultiaddr: /ip4/127.0.0.1/tcp/8337
  ```
- Save the file with `Ctrl+O`, and exit with `Ctrl+X`.
- Restart the node


## Get Node Peer ID and Balance

- Get your Node Peer ID:

  ```bash
  cd ~/ceremonyclient/node
  GOEXPERIMENT=arenas go run ./... -peer-id
  ```
- Get your $QUIL balance (`gRPC` must be configured):

  ```bash
  cd ~/ceremonyclient/node
  GOEXPERIMENT=arenas go run ./... -balance
  ```

## Save your keys to your local computer
- You will want to save `config.yml` and `keys.yml` in a safe place. To copy them from your VPS to your local computer run the following commands from your local terminal, making sure to replace `VPS_IP_ADDRESS` with that of the VPS you're trying to copy from, and `FULL_DIRECTORY_PATH_WHERE_YOU_WANT_TO_SAVE_THE_FILE` with your prefered directory to which you'd like to save:
- saving `config.yml`
  ```bash
  scp -rp root@VPS_IP_ADDRESS:/root/ceremonyclient/node/.config/config.yml /FULL_DIRECTORY_PATH_WHERE_YOU_WANT_TO_SAVE_THE_FILE
  ```
- saving `keys.yml`
  ```bash
  scp -rp root@VPS_IP_ADDRESS:/root/ceremonyclient/node/.config/keys.yml /FULL_DIRECTORY_PATH_WHERE_YOU_WANT_TO_SAVE_THE_FILE
  ```
