# Install and configure the Docker

In this lab you will install and configure Docker on node0 and node1.

## Configure the Docker Engine

### node0
Open a second Terminal/Console

```
gcloud compute ssh node0
```
If the node ask for a password, then you connected too quick after booting the node. Press `^c` and try again. For troubleshooting, logs can be found in the [GCE console](https://console.cloud.google.com/compute/instances), then click on the name of a node, scroll all the way down and click on _Serial console output_.

### Create the docker systemd unit file

```
curl -O https://storage.googleapis.com/configs.kuar.io/docker.service
```

Configure the docker unit file

Usage of `--daemon` is deprecated and set the `--bip` flag to `10.200.0.1/24`:

```
sed -i -e "s/--daemon/daemon/g;" docker.service
sed -i -e "s/BRIDGE_IP/10.200.0.1\/24/g;" docker.service
```

Review the docker unit file.

```
cat docker.service
```

Copy the docker unit file into place.

```
sudo mv docker.service /etc/systemd/system/docker.service
```

Start docker:

```
sudo systemctl daemon-reload
sudo systemctl enable docker
sudo systemctl start docker
```

#### Verify

```
ip addr show docker0
```
Notice that node`0` gives out ip-addresses in the range 10.200.`0`.0/24
```
docker version
```

### node1
Open a third terminal/console

```
gcloud compute ssh node1
```

### Create the docker systemd unit file

```
curl -O https://storage.googleapis.com/configs.kuar.io/docker.service
```

Configure the docker unit file

Usage of `--daemon` is deprecated and set the `--bip` flag to `10.200.1.1/24`:

```
sed -i -e "s/--daemon/daemon/g;" docker.service
sudo sed -i -e "s/BRIDGE_IP/10.200.1.1\/24/g;" docker.service
```

Review the docker unit file.

```
cat docker.service
```

Copy the docker unit file into place.

```
sudo mv docker.service /etc/systemd/system/docker.service
```

Start docker:

```
sudo systemctl daemon-reload
sudo systemctl enable docker
sudo systemctl start docker
```

#### Verify

```
ip addr show docker0
```
Notice that node`1` gives out ip-addresses in the range 10.200.`1`.0/24 which doesn't overlap with node0.

```
docker version
```
