# Configuring the Network

In this lab you will configure the network between node0 and node1 to ensure cross host connectivity. You will also ensure containers can communicate across hosts and reach the internet.

## Create network routes between Docker hosts.

### laptop
Kubernetes uses standard L3 networking. Tell the Google global router where to find 10.200.0.0 and 10.200.1.0
![](images/GCE-routing.png?raw=true)
```
gcloud compute routes create default-route-10-200-0-0-24 \
  --destination-range 10.200.0.0/24 \
  --next-hop-instance node0
```
```
gcloud compute routes create default-route-10-200-1-0-24 \
  --destination-range 10.200.1.0/24 \
  --next-hop-instance node1
```

```
gcloud compute routes list
```

## Allow external access to the API server secure port

```
gcloud compute firewall-rules create default-allow-kubernetes-secure \
  --allow tcp:6443 \
  --source-ranges 0.0.0.0/0
```

## Allow add-ons to query the API server
Internal clients are allowed access to the API server _without SSL_. This firewall rule is only recommended if you trust your internal clients and _should be executed with care_.
```
gcloud compute firewall-rules create default-allow-local-api \
  --allow tcp:8080 \
  --source-ranges 10.200.0.0/16
```


## Getting Containers Online

Create an `iptables` rule to masquerade traffic that is destined (`! -d`) for IPs outside the GCE project network (10.0.0.0/8). Masquerade (aka SNAT) - to make it seem as if packets came from the Node itself - and allow connectivity to Internet. More on [CGE networking here](http://kubernetes.io/docs/admin/networking/#google-compute-engine-gce).

### node0

```
gcloud compute ssh node0
```

```
sudo iptables -t nat -A POSTROUTING ! -d 10.0.0.0/8 -o ens4v1 -j MASQUERADE
sudo iptables -t nat -L -n -v
```

### node1

```
gcloud compute ssh node1
```

```
sudo iptables -t nat -A POSTROUTING ! -d 10.0.0.0/8 -o ens4v1 -j MASQUERADE
```

> __TIP__: persist after reboots using
```
sudo su -
cat > /etc/systemd/system/iptables.service <<-EOF
    [Unit]
    Description=Packet Filtering Framework
    DefaultDependencies=no
    After=systemd-sysctl.service
    Before=sysinit.target
    [Service]
    Type=oneshot
    ExecStart=/sbin/iptables-restore /var/lib/iptables/rules-save
    ExecReload=/sbin/iptables-restore /var/lib/iptables/rules-save
    # ExecStop=/etc/iptables/flush-iptables.sh
    RemainAfterExit=yes
    [Install]
    WantedBy=multi-user.target
EOF
cat >/var/lib/iptables/rules-save <<-EOF
	*nat
	:POSTROUTING ACCEPT [0:0]
	-A POSTROUTING ! -d 10.0.0.0/8 -o ens4v1 -j MASQUERADE
	COMMIT
EOF
systemctl daemon-reload
# systemctl enable iptables
systemctl enable iptables.service
systemctl start iptables.service
systemctl status iptables.service
reboot # for Test
```

> verify SNAT persists
```
gcloud compute ssh node # 0 or 1
sudo iptables -t nat -L -v
```

## Validating Cross Host Container Networking

### Terminal 1

```
gcloud compute ssh node0
```
The Linux node has a IP forwarding enabled and routes all 10.200.1.0 traffic to the `docker0` bridge.  
```
sysctl net.ipv4.ip_forward
netstat -rn
```
Containers will get an ip-address from docker0  
```
docker run -t -i --rm busybox /bin/sh
ip -f inet addr show eth0
```

### Terminal 2

```
gcloud compute ssh node1
```

```
docker run -t -i --rm busybox /bin/sh
```

```
ping -c 3 10.200.0.2
```

```
ping -c 3 google.com
```

Exit both busybox instances.
