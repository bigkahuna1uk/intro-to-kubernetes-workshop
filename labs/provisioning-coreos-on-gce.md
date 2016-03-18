# Provisioning CoreOS on Google Compute Engine

In this lab you will provision two GCE instances running CoreOS.

## Provision 2 GCE instances

### Provision CoreOS using the gcloud CLI

#### node0

```
gcloud compute instances create node0 \
 --image-project coreos-cloud \
 --image coreos-beta-899-11-0-v20160317 \
 --boot-disk-size 200GB \
 --machine-type n1-standard-1 \
 --can-ip-forward
```

#### node1

```
gcloud compute instances create node1 \
 --image-project coreos-cloud \
 --image coreos-beta-899-11-0-v20160317 \
 --boot-disk-size 200GB \
 --machine-type n1-standard-1 \
 --can-ip-forward
```

#### Verify

```
gcloud compute instances list
```
> __TIP__: to save on GCE bills you can stop the servers and restart them another time.
```
gcloud compute instances stop node0 node1
gcloud compute instances start node0 node1
```
To make iptables rules persists after reboots see TIP at end of configure-networking.md
