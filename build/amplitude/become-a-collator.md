# Become a collator

This guide will describe how to install and run an Amplitude Collator node on docker. The following link will contain the location of the Docker images:

[https://hub.docker.com/repository/docker/pendulumchain/pendulum-collator](https://hub.docker.com/repository/docker/pendulumchain/pendulum-collator)

### Hardware and Software requirements

In order to be able to run a collator node you will need at least a machine with the following specs:

* 4 CPU cores
* 16 GB of RAM
* 250 GB of Disk space
* Ubuntu 20.04.5 LTS

:warning: Other operating systems apart from Debian and Ubuntu based distributions were not tested yet, but you can always try at your own risk.

### Step-by-step installation

We assume that your Ubuntu server is a fresh install, it does have an external IP address that has direct access to the internet.

Apply the latest patches

```bash
sudo apt update
sudo apt upgrade -y
```

Follow the steps below to install docker on your ubuntu server:

```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common git binutils -y
curl -fsSL <https://download.docker.com/linux/ubuntu/gpg> | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] <https://download.docker.com/linux/ubuntu> `lsb_release -cs` stable"
sudo apt update
sudo apt-cache policy docker-ce 
sudo apt install docker-ce -y
sudo systemctl status docker
```

Configure your local firewall to allow communication for the required ports

```bash
sudo ufw allow 8844/tcp
sudo ufw allow 30335/tcp
sudo ufw allow 9955/tcp
sudo ufw allow 30334/tcp
sudo ufw allow https
sudo ufw enable
```

Download chain specifications

```bash
sudo mkdir -p /data/
sudo wget -P /data <https://raw.githubusercontent.com/paritytech/polkadot/master/node/service/chain-specs/kusama.json>
sudo wget -P /data <https://raw.githubusercontent.com/pendulum-chain/pendulum/main/res/amplitude-spec-raw.json>
```

Install the application using docker. This will name the docker container the same as the hostname of the Ubuntu server. If you wish to change that, replace the **$(hostname)** parameter with a name of your choice.

```bash
sudo docker pull pendulumchain/pendulum-collator:v0.9.24
docker run --name $(hostname) --restart unless-stopped -d -v /data:/data -it -p 30335:30335 -p 9955:9955 -p 30334:30334 -p 8844:8844 pendulumchain/pendulum-collator:v0.9.24 --collator --allow-private-ipv4 --unsafe-ws-external --rpc-cors all --rpc-external --rpc-methods Unsafe --force-authoring --enable-offchain-indexing=TRUE --ws-port 8844 --ws-max-connections 200 --port 30335 --rpc-port 9955 --chain /data/amplitude-spec-raw.json --execution=wasm -- --port 30334 --chain /data/kusama.json --database=RocksDb --execution=wasm -d /data --unsafe-pruning --pruning=256
```

### Verify your installation

Run **`docker ps`** command to verify if the collator is running properly. You should see the following output:

```bash
# docker ps
CONTAINER ID   IMAGE                                     COMMAND                  CREATED          STATUS          PORTS                                                                                                                                                                   NAMES
3212d72e1291   pendulumchain/pendulum-collator:v0.9.24   "tini -- /usr/local/…"   10 minutes ago   Up 10 minutes   0.0.0.0:8844->8844/tcp, :::8844->8844/tcp, 0.0.0.0:9935->9935/tcp, :::9935->9935/tcp, 0.0.0.0:30334-30335->30334-30335/tcp, :::30334-30335->30334-30335/tcp, 9945/tcp   yourhostname
```

The **/data** directory should start to get populated with data and should have a structure:

```bash
drwxr-xr-x 3 root root    6144 Sep 28 11:09 chains
-rw-r--r-- 1 root root 2861651 Oct  4 16:36 kusama.json
-rw-r--r-- 1 root root 1326607 Oct  4 16:36 amplitude-spec-raw.json
```

### What happens next?

Now your collator will start syncing both **Kusama** and **Amplitude** chains. Kusama syncing may take several days to complete due to the size of the Kusama database, this is normal and expected. You can follow the progress in the log files using the **docker logs -f $(hostname)** command.

```bash
2022-09-28 19:02:26 assembling new collators for new session 45 at #52800
2022-09-28 19:02:30 ✨ Imported #14652094 (0x7505…a7d8)
2022-09-28 19:02:30 💤 Idle (49 peers), best: #14652094 (0x7505…a7d8), finalized #14652090 (0xbc43…9e95), ⬇ 488.0kiB/s ⬆ 425.0kiB/s
2022-09-28 19:02:31 ⚙️  Syncing 224.8 bps, target=#181974 (12 peers), best: #53857 (0x2f61…0009), finalized #0 (0xccea…1aaf), ⬇ 1.4MiB/s ⬆ 2.1kiB/s
```

When the node is syncing, you can see the block it is trying to sync up to. The “**best**” section will tell you what block the sync actually is at, and the “**target**” where it needs to get to.

### How to speed up the Kusama chain syncing process?

As we previously pointed out, syncing the **Kusama** chain could take many days. Luckily enough, there is a solution for that provided by [Polkashots](https://polkashots.io/). Follow the procedure here:[https://ksm-rocksdb.polkashots.io/](https://ksm-rocksdb.polkashots.io/) to try and speed up your sync procedure.

:bulb: Polkashots is a 3rd party service provider and in no way affiliated with Pendulum/Amplitude