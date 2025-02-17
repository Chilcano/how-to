# Raspberry Pi - Getting started

## A. Installing and configuring Ubuntu OS

### 1. Copy an OS image in your SD card

Nowadays, you have many mature and very well documented ways to copy many flavours of Linux in your Raspberry Pi.
You have:

1. The old school `dd` Linux command. Use it if you are confortable using a Linux Terminal.

List the SD cards:
```sh
sd_cards_list=$(lsblk --nodeps -n -o name -I8)

echo ${sd_cards_list}

sda sdb sdc
```

Run `dd`:
```sh
sudo dd bs=1M if=/path/to/raspberrypi/image of=/dev/sdcardname status=progress conv=fsync
```

2. Balena Etcher or __Raspberry Pi Imager__. Yes, there are many and you can use any of them. 

Here you have 2 good guides:
- https://www.raspberrypi.com/documentation/computers/getting-started.html
- https://www.makeuseof.com/tag/install-operating-system-raspberry-pi/

Once copied your Raspbian, Raspberry Pi OS or Ubuntu OS image in your SD Card, the next steps will explain how to enable remote SSH access, configure the physical or wireless network interface of your Raspberry Pi. 

We are going to use Raspberry Pi Imager, it can install Rasbian OS, Ubuntu or any proper custom image for Raspberry Pi and bootstrap initial configuration like enable `ssh` or setup a `hostname`.
```sh
$ sudo apt -y install rpi-imager
```

### 2. Bootstrap an initial configuration

You can use this [bash script to bootstrap an initial configuration](../src/bootstrap_config_rpi.sh) for your pre-installed Ubuntu, Raspbian, Debian or Raspberry Pi OS. Specifically, the bash script will enable SSH and will enable and configure WIFI before using to boot the SD Card. Download it, set it up as executable:
```sh
wget -qN https://raw.githubusercontent.com/chilcano/how-tos/main/src/bootstrap_config_rpi.sh
chmod +x bootstrap_config_rpi.sh
```
and run it.
```sh
. bootstrap_config_rpi.sh --wifi=enable
```

Or this single command:
```sh
source <(curl -s https://raw.githubusercontent.com/chilcano/how-tos/main/src/bootstrap_config_rpi.sh) --wifi=enable
```

### 3. Insert SD Card and boot your RPi

You can connect your Raspberry Pi to:
1. To computer directly through Ethernet port or USB-to-Eth Adapter. I've tested this with an Ubuntu Laptop without Ethernet port. Ubuntu detects inmediatelly and assigns an internal IP Address.
2. To your Network LAN using a Ethernet cable directly.
3. To your Wireless LAN. You need to pre-configure your WIFI when burning the Image into your Raspberry Pi.

The next steps help you how to do any scenario.

### 4. RPi connected directly to Ubuntu Laptop

__Observations:__
> Unfortunately, this headless method didn't work to me when using `2024-07-04-raspios-bookworm-arm64` and worked previous versions.


* 4.1. Connect Raspberry Pi to Laptop with Ethernet.

* 4.2. Go to Settings > Network > Wired.

![](img/code-server-headless-rpi-ubuntu-64bits-network-connection-01.png)
![](img/code-server-headless-rpi-ubuntu-64bits-network-connection-02.png)

* 4.4. Navigate to `IPv4` option and select `Shared to other computers`.

* 4.4. Open Terminal and type next command (`$ ip a s`) to get the IP address for the `enx<MAC-ADDRESS>` or `enp<STRING>` Network Interface. Also you can see it in `Wired Settings > IPv4`. 

```sh
ip a s
```

You will see all network interfaces listed:
```sh
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: wlp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 90:78:41:13:cb:e9 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.152/24 brd 192.168.1.255 scope global dynamic noprefixroute wlp2s0
       valid_lft 72207sec preferred_lft 72207sec
    inet6 fe80::89c7:1f89:b09b:70ab/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:36:d4:eb:b2 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
4: br-e62192fa6741: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:c1:fc:b6:17 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-e62192fa6741
       valid_lft forever preferred_lft forever
6: enx8cae4cf9014c: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 8c:ae:4c:f9:01:4c brd ff:ff:ff:ff:ff:ff
    inet 10.42.0.1/24 brd 10.42.0.255 scope global noprefixroute enx8cae4cf9014c
       valid_lft forever preferred_lft forever
    inet6 fe80::aee5:aac2:28ef:e868/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

* 4.5. Install `nmap` 7.9x (7.8 has a bug). The `nmap` will scan and identify the assigned RPi's IP address to connected Laptop.

```sh
sudo apt install snapd
sudo snap install nmap
sudo snap connect nmap:network-control
``` 
Now, you are ready to run `nmap`.

* 4.6. With that IP address (`enx8cae4cf9014c with 10.42.0.1/24`), run the next command to scan all active IP addresses under the `10.42.0.0/24` network. Generally you will have 2 IP addresses, one of they will be the RPi's IP address.

```sh
nmap -sn 10.42.0.0/24
```

You should see something like this:
```sh
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-04 00:28 CET
Nmap scan report for inti (10.42.0.1)
Host is up (0.00027s latency).
Nmap scan report for 10.42.0.159
Host is up (0.00057s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 2.40 seconds
```

Run it with `sudo` to get hostnames and MAC:
```sh
sudo nmap -sn 10.42.0.0/24
```

You will see hostnames now:
```sh
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-09 17:57 CET
Nmap scan report for 10.42.0.159
Host is up (0.00068s latency).
MAC Address: B8:27:EB:1B:CF:C8 (Raspberry Pi Foundation)
Nmap scan report for inti (10.42.0.1)
Host is up.
Nmap done: 256 IP addresses (2 hosts up) scanned in 9.25 seconds
```

* 4.7. SSH to Raspberry Pi from the Laptop Terminal: 

```sh
ssh pi@<ip-of-raspberry-pi>       ## Pwd: raspberry, if you used Raspbian or Raspberry OS
ssh ubuntu@<ip-of-raspberry-pi>   ## Pwd: ubuntu, if you used Ubuntu OS
```
In our case the ssh command is:
```sh
ssh pi@10.42.0.159
```

### 5. RPi connected directly to same Ubuntu Laptop's LAN 

* 5.1. Getting the Ubuntu Laptop's IP address.

```sh
hostname -I
```

```sh
192.168.1.152 172.18.0.1 172.17.0.1
```

* 5.2. Getting the Raspberry Pi IP address using `nmap` (version 7.9.x).

The `nmap` version 7.8 doesn't work, we recommend install fixed version (above 7.9.x). [Here we explain how to do that](nmap_commands.md).

```sh
sudo nmap -sn 192.168.1.0/24
```

```sh
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-09 22:56 CET
Nmap scan report for 192.168.1.160
Host is up (0.0040s latency).
MAC Address: B8:27:EB:1B:CF:C8 (Raspberry Pi Foundation)
Nmap scan report for inti (192.168.1.152)
Host is up.
Nmap done: 256 IP addresses (2 hosts up) scanned in 2.46 seconds
```

* 5.3. SSH to Raspberry Pi from the Laptop Terminal: 

```sh
ssh pi@<ip-of-raspberry-pi>       ## Pwd: raspberry, if you used Raspbian or Raspberry OS
ssh ubuntu@<ip-of-raspberry-pi>   ## Pwd: ubuntu, if you used Ubuntu OS
```

### 6. RPi connected directly to same Ubuntu Laptop's Wireless LAN

* 6.1. Plug the already configured SD card into the Raspberry Pi, turn on. After 2 to 3 minutes, turn off and turn on. This process will force your Raspberry Pi connects to your pre-configured WIFI configuration.

* 6.2. Getting the IP address. The process is similar to above.

```sh
sudo nmap -sn 192.168.1.0/24
```

```sh
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-17 20:09 CET
Nmap scan report for 192.168.1.162
Host is up (0.026s latency).
MAC Address: B8:27:EB:33:B7:BE (Raspberry Pi Foundation)
Nmap scan report for kipu (192.168.1.152)
Host is up.
Nmap done: 256 IP addresses (8 hosts up) scanned in 2.07 seconds
```

* 6.3. SSH to Raspberry Pi from the Laptop Terminal: 

```sh
ssh pi@<ip-of-raspberry-pi>       ## Pwd: raspberry, if you used Raspbian or Raspberry OS
ssh ubuntu@<ip-of-raspberry-pi>   ## Pwd: ubuntu, if you used Ubuntu OS
```

## B. Installing and configuring Rasbian OS