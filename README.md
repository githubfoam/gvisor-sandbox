# gvisor-sandbox

Install Docker
~~~~
vagrant@gvisor-sandbox:~$ hostnamectl
   Static hostname: gvisor-sandbox
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 081f3e07ae294d0ca034bf295f7cfa36
           Boot ID: 9719dc8202734c9c94b4c78e80dd614c
    Virtualization: oracle
  Operating System: Ubuntu 18.04.1 LTS
            Kernel: Linux 4.15.0-29-generic
      Architecture: x86-64

~~~~

Install gVisor
~~~~
vagrant@gvisor-sandbox:~$ hostnamectl
   Static hostname: gvisor-sandbox
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 081f3e07ae294d0ca034bf295f7cfa36
           Boot ID: 9719dc8202734c9c94b4c78e80dd614c
    Virtualization: oracle
  Operating System: Ubuntu 18.04.1 LTS
            Kernel: Linux 4.15.0-29-generic
      Architecture: x86-64

DIST=release

sudo apt-get update && \
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

"curl -fsSL https://gvisor.dev/archive.key | sudo apt-key add -"    

sudo add-apt-repository "deb https://storage.googleapis.com/gvisor/releases ${DIST} main"


 vagrant@gvisor-sandbox:~$ sudo add-apt-repository "deb https://storage.googleapis.com/gvisor/releases ${DIST} main"
 Hit:1 http://archive.ubuntu.com/ubuntu bionic InRelease
 Hit:2 http://archive.ubuntu.com/ubuntu bionic-updates InRelease
 Hit:3 http://archive.ubuntu.com/ubuntu bionic-backports InRelease
 Hit:4 http://security.ubuntu.com/ubuntu bionic-security InRelease
 Hit:5 https://download.docker.com/linux/ubuntu bionic InRelease
 Get:6 https://storage.googleapis.com/gvisor/releases release InRelease [2,712 B]
 Get:7 https://storage.googleapis.com/gvisor/releases release/main amd64 Packages [516 B]
 Fetched 3,228 B in 2s (1,993 B/s)
 Reading package lists... Done
 W: Conflicting distribution: https://storage.googleapis.com/gvisor/releases release InRelease (expected release but got )
 N: Skipping acquire of configured file 'main/binary-i386/Packages' as repository 'https://storage.googleapis.com/gvisor/releases release InRelease' doesn't support architecture 'i386'
 vagrant@gvisor-sandbox:~$ sudo add-apt-repository "deb https://storage.googleapis.com/gvisor/releases release main"

 sudo apt-get update && sudo apt-get install -y runsc

 vagrant@gvisor-sandbox:~$ sudo runsc install
2020/02/27 16:21:27 Added runtime "runsc" with arguments [] to "/etc/docker/daemon.json".

sudo systemctl restart docker
sudo docker run --runtime=runsc --rm hello-world

vagrant@gvisor-sandbox:~$ sudo docker run --runtime=runsc -it ubuntu uname -a
Linux c6899ea0283a 4.4.0 #1 SMP Sun Jan 10 15:06:54 PST 2016 x86_64 x86_64 x86_64 GNU/Linux

vagrant@gvisor-sandbox:~$ cat /etc/docker/daemon.json
{
    "runtimes": {
        "runsc": {
            "path": "/usr/bin/runsc"
        }
    }
}vagrant@gvisor-sandbox:~$


~~~~

OCI compatible container
~~~~
vagrant@gvisor-sandbox:~$ hostnamectl
   Static hostname: gvisor-sandbox
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 081f3e07ae294d0ca034bf295f7cfa36
           Boot ID: 9719dc8202734c9c94b4c78e80dd614c
    Virtualization: oracle
  Operating System: Ubuntu 18.04.1 LTS
            Kernel: Linux 4.15.0-29-generic
      Architecture: x86-64

mkdir bundle && cd bundle && mkdir rootfs

vagrant@gvisor-sandbox:~/bundle$ sudo docker export $(sudo docker create hello-world) | sudo tar -xf - -C rootfs
vagrant@gvisor-sandbox:~/bundle$ sudo runsc spec
vagrant@gvisor-sandbox:~/bundle$ sed -i 's;"sh";"/hello";' config.json
vagrant@gvisor-sandbox:~/bundle$ sudo runsc run hello

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

vagrant@gvisor-sandbox:~/bundle$

~~~~

Using CNI
~~~~
vagrant@gvisor-sandbox:~$ hostnamectl
   Static hostname: gvisor-sandbox
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 081f3e07ae294d0ca034bf295f7cfa36
           Boot ID: 9719dc8202734c9c94b4c78e80dd614c
    Virtualization: oracle
  Operating System: Ubuntu 18.04.1 LTS
            Kernel: Linux 4.15.0-29-generic
      Architecture: x86-64

 sudo mkdir -p /opt/cni/bin
 wget https://github.com/containernetworking/plugins/releases/download/v0.8.3/cni-plugins-linux-amd64-v0.8.3.tgz
 sudo tar -xvf cni-plugins-linux-amd64-v0.8.3.tgz -C /opt/cni/bin/
 sudo mkdir -p /etc/cni/net.d

vagrant@gvisor-sandbox:~$ sudo tar -xvf cni-plugins-linux-amd64-v0.8.3.tgz -C /opt/cni/bin/
./
./flannel
./ptp
./host-local
./firewall
./portmap
./tuning
./vlan
./host-device
./bandwidth
./sbr
./static
./dhcp
./ipvlan
./macvlan
./loopback
./bridge
vagrant@gvisor-sandbox:~$ sudo mkdir -p /etc/cni/net.d
vagrant@gvisor-sandbox:~$
vagrant@gvisor-sandbox:~$ sudo sh -c 'cat > /etc/cni/net.d/10-bridge.conf << EOF
> {
>   "cniVersion": "0.4.0",
>   "name": "mynet",
>   "type": "bridge",
>   "bridge": "cni0",
>   "isGateway": true,
>   "ipMasq": true,
>   "ipam": {
>     "type": "host-local",
>     "subnet": "10.22.0.0/16",
>     "routes": [
>       { "dst": "0.0.0.0/0" }
>     ]
>   }
> }
> EOF'
vagrant@gvisor-sandbox:~$ cat /etc/cni/net.d
cat: /etc/cni/net.d: Is a directory
vagrant@gvisor-sandbox:~$ cat /etc/cni/net.d/10-bridge.conf
{
  "cniVersion": "0.4.0",
  "name": "mynet",
  "type": "bridge",
  "bridge": "cni0",
  "isGateway": true,
  "ipMasq": true,
  "ipam": {
    "type": "host-local",
    "subnet": "10.22.0.0/16",
    "routes": [
      { "dst": "0.0.0.0/0" }
    ]
  }
}
vagrant@gvisor-sandbox:~$ sudo sh -c 'cat > /etc/cni/net.d/99-loopback.conf << EOF
> {
>   "cniVersion": "0.4.0",
>   "name": "lo",
>   "type": "loopback"
> }
> EOF'
vagrant@gvisor-sandbox:~$ cat /etc/cni/net.d/99-loopback.conf
{
  "cniVersion": "0.4.0",
  "name": "lo",
  "type": "loopback"
}
vagrant@gvisor-sandbox:~$



vagrant@gvisor-sandbox:~$ export CNI_PATH=/opt/cni/bin
vagrant@gvisor-sandbox:~$ export CNI_CONTAINERID=$(printf '%x%x%x%x' $RANDOM $RANDOM $RANDOM $RANDOM)
vagrant@gvisor-sandbox:~$ export CNI_COMMAND=ADD
vagrant@gvisor-sandbox:~$ export CNI_NETNS=/var/run/netns/${CNI_CONTAINERID}
vagrant@gvisor-sandbox:~$ sudo ip netns add ${CNI_CONTAINERID}
vagrant@gvisor-sandbox:/tmp/bundle$ ls /var/run/netns/${CNI_CONTAINERID}
/var/run/netns/8d91e253e813977
vagrant@gvisor-sandbox:/tmp/bundle$ sudo ip netns list
8d91e253e813977

vagrant@gvisor-sandbox:~$ export CNI_IFNAME="eth0"
vagrant@gvisor-sandbox:~$ sudo -E /opt/cni/bin/bridge < /etc/cni/net.d/10-bridge.conf
{
    "cniVersion": "0.4.0",
    "interfaces": [
        {
            "name": "cni0",
            "mac": "8e:61:ff:b1:a8:16"
        },
        {
            "name": "veth3b042c76",
            "mac": "32:34:93:7d:a0:77"
        },
        {
            "name": "eth0",
            "mac": "ce:aa:e7:b9:2c:52",
            "sandbox": "/var/run/netns/47741d1c1aee22d9"
        }
    ],
    "ips": [
        {
            "version": "4",
            "interface": 2,
            "address": "10.22.0.2/16",
            "gateway": "10.22.0.1"
        }
    ],
    "routes": [
        {
            "dst": "0.0.0.0/0"
        }
    ],
    "dns": {}
}vagrant@gvisor-sandbox:~$ export CNI_IFNAME="lo"
vagrant@gvisor-sandbox:~$ sudo -E /opt/cni/bin/loopback < /etc/cni/net.d/99-loopback.conf
{
    "cniVersion": "0.4.0",
    "interfaces": [
        {
            "name": "lo",
            "mac": "00:00:00:00:00:00",
            "sandbox": "/var/run/netns/47741d1c1aee22d9"
        }
    ],
    "ips": [
        {
            "version": "4",
            "interface": 0,
            "address": "127.0.0.1/8"
        },
        {
            "version": "6",
            "interface": 0,
            "address": "::1/128"
        }
    ],
    "dns": {}
}vagrant@gvisor-sandbox:~$


IP address assigned to the sandbox

vagrant@gvisor-sandbox:~$ POD_IP=$(sudo ip netns exec ${CNI_CONTAINERID} ip -4 addr show eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')
vagrant@gvisor-sandbox:~$ echo $POD_IP
10.22.0.2

Create the OCI Bundle
network namespace is created and configured
create the OCI bundle for the container.
run a simple python webserver that connects to via the IP address assigned to the bridge CNI plugin.

vagrant@gvisor-sandbox:~$ cd /tmp && sudo mkdir -p bundle && cd bundle
vagrant@gvisor-sandbox:/tmp/bundle$ sudo docker export $(sudo docker create python) | sudo tar --same-owner -pxf - -C rootfs

vagrant@gvisor-sandbox:/tmp/bundle$ sudo mkdir -p rootfs/var/www/html
vagrant@gvisor-sandbox:/tmp/bundle$ sudo sh -c 'echo "Hola el mundo!" > rootfs/var/www/html/index.html'
vagrant@gvisor-sandbox:/tmp/bundle$ which runsc
/usr/bin/runsc
vagrant@gvisor-sandbox:/tmp/bundle$ sudo /usr/bin/runsc spec
vagrant@gvisor-sandbox:/tmp/bundle$ sudo sed -i 's;"sh";"python", "-m", "http.server";' config.json
vagrant@gvisor-sandbox:/tmp/bundle$ sudo sed -i "s;\"cwd\": \"/\";\"cwd\": \"/var/www/html\";" config.json
vagrant@gvisor-sandbox:/tmp/bundle$ sudo sed -i "s;\"type\": \"network\";\"type\": \"network\",\n\t\t\t\t\"path\": \"/var/run/netns/${CNI_CONTAINERID}\";" config.json
vagrant@gvisor-sandbox:/tmp/bundle$

vagrant@gvisor-sandbox:/tmp/bundle$ sudo runsc run -detach ${CNI_CONTAINERID}
vagrant@gvisor-sandbox:/tmp/bundle$ curl http://${POD_IP}:8000/
Hola el mundo!

cleanup

vagrant@gvisor-sandbox:/tmp/bundle$ sudo runsc kill ${CNI_CONTAINERID}
vagrant@gvisor-sandbox:/tmp/bundle$ sudo runsc delete ${CNI_CONTAINERID}
vagrant@gvisor-sandbox:/tmp/bundle$ export CNI_COMMAND=DEL
vagrant@gvisor-sandbox:/tmp/bundle$ export CNI_IFNAME="lo"
vagrant@gvisor-sandbox:/tmp/bundle$ sudo -E /opt/cni/bin/loopback < /etc/cni/net.d/99-loopback.conf
vagrant@gvisor-sandbox:/tmp/bundle$ export CNI_IFNAME="eth0"
vagrant@gvisor-sandbox:/tmp/bundle$ sudo -E /opt/cni/bin/bridge < /etc/cni/net.d/10-bridge.conf
vagrant@gvisor-sandbox:/tmp/bundle$ sudo ip netns delete ${CNI_CONTAINERID}
~~~~

~~~~
gVisor can be used with Docker, Kubernetes, or directly using runsc with crafted OCI spec for your container. Use the links below to see detailed instructions for each of them:

    Docker: The quickest and easiest way to get started.
    Kubernetes: Isolate Pods in your K8s cluster with gVisor.
    OCI: Expert mode. Customize gVisor for your environment.

https://gvisor.dev/docs/user_guide/quick_start/

Docker Quick Start
https://gvisor.dev/docs/user_guide/docker/
Installation
    Note: gVisor supports only x86_64 and requires Linux 4.14.77+ (older Linux).
https://gvisor.dev/docs/user_guide/install/

OCI
https://gvisor.dev/docs/user_guide/quick_start/oci/
The OCI currently contains two specifications: the Runtime Specification (runtime-spec) and the Image Specification (image-spec).
https://www.opencontainers.org/

This tutorial will show you how to set up networking for a gVisor sandbox using the Container Networking Interface (CNI).
https://gvisor.dev/docs/tutorials/cni/
~~~~
