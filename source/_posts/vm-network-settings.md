---
title: Network settings for virtualbox
date: 2016-11-21 16:49:04
tags:
  - vm
  - network
  - linux
---

*※ 2017/09/20 更新 Ubuntu 16.04*

- VM: CentOS 6.5 / Ubuntu 16.04
- Host: Windows7

<!-- more -->

## VM 設定

- 網卡設定
   - 第一張設 **NAT**
   - 第二張設 **橋接介面卡**

### CentOS 6.5

1. 確認 VM 網卡設定 
    ```shell
    $ ifconfig
    ```
   - 確認兩張網卡都有開 `eth0` / `eth1`
   - 沒有的話，就啟動網卡
   ```shell
   $ ifconfig {eth0|eth1} {up|down}
   ```
   
2. VM網卡設定
    
    ```shell
    $ vim /etc/sysconfig/network-scripts/ifcfg-{eth0|eth1}
    ```

    - eth0 (NAT) 基本上原本的設定就好，不用改
    ```config
    DEVICE=eth0
    BOOTPROTO=dhcp
    HWADDR=08:00:27:92:0E:1F #your mac address
    ONBOOT=yes
    TYPE=Ethernet
    ```
    - eth1 (橋接)
    ```config
    DEVICE=eth1
    #BOOTPROTO=none
    #BROADCAST=192.168.XXX.255
    HWADDR=08:00:27:79:90:9d #your mac address
    IPADDR=192.168.XXX.VVV
    NETMASK=255.255.255.0
    #NETWORK=192.168.XXX.0
    ONBOOT=yes
    #GATEWAY=192.168.XXX.1
    TYPE=Ethernet
    ```
    
3. 設定完記得重啟
```shell
$ /etc/init.d/network restart
```

4. 檢查連線 & you're good to go!

#### 其他設定

- 關閉防火牆

```shell
$ service iptables stop
```
- 設定開機時不會開起防火牆

```
$ chkconfig iptables off
```

### Ubuntu 16.04

1. 先確認有哪些網卡

```
$ ip addr
```

2. 進入 `/etc/network/interfaces` 修改檔案

```
$ sudo vim /etc/network/interfaces
```

```yaml
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto enp0s3
iface enp0s3 inet dhcp

# 以下為 橋接網卡 主要新增的部分 (上面都是預設的，不用改)
auto enp0s8
iface enp0s8 inet static
address 192.168.13.14
netmask 255.255.255.0
```

3. 重啟連線

```
$ sudo /etc/init.d/networking restart
```

4. Check and done!

```
$ ping 8.8.8.8
```

## 本機設定

1. 確定有加網路區段 (Windows)
    > 區域連線 > IPv4 > 進階 > IP設定

   增加和VM同網段的IP位址 `192.168.XXX.LLL`
   
2. 檢查是否連得到 VM
```shell
$ ping 192.168.XXX.VVV
```

## 備註

（以下筆記）

VMware Player 的 host OS 和 guest OS 之間有三種網路型態：

### Bridged Networking

在這種網路型態之下，guest OS 是透過一個 virtual bridge 和 host OS 所在的 Ethernet 相連，請參考 VMware 的官方 [示意圖](https://www.vmware.com/support/ws5/doc/ws_net_configurations_bridged.html)。

對 於與 host OS 同在一個 Ethernet 上的機器來說，guest OS 和 host OS 是兩台獨立的電腦，都可以透過同一個 Ethernet 介面連接，並無法分辨出這兩個 OS 其實是在同一台機器上執行。甚至當 Ethernet 連線出問題時，guest OS 和 host OS 也不能互通 (即使是在同一台機器之內)。

換句話說，當你把接到 host OS 的網路線拔掉時，這兩個 OS 就無法溝通。因此，想把 VMware Player 灌在 notebook 上帶著跑的人，這種網路型態是不太合適的。

### Host-Only Networking

在 這種網路型態之下，guest OS 和 host OS 是在一個與世隔絕的虛擬網路上。此虛擬網路有一個 DHCP server，可以分配 IP address 給 guest OS 和 host OS (分配給一個虛擬的介面)。因此，guest OS 和 host OS 可以互通。請參考 VMware 的官方[示意圖](https://www.vmware.com/support/ws5/doc/ws_net_configurations_hostonly.html)。

對 於與 host OS 同在一個 Ethernet 上的機器來說，guest OS 是看不見的。guest OS 對外聯繫的唯一管道就是 host OS。因此，guest OS 若想連上外部網路或 Internet，就必須在 host OS 安裝 routing 或 NAT 的服務。

這種架構不會受到實體網路的影響，即使把網路線拔掉，host OS 和 guest OS 還是可以互通。

### Network Address Translation (NAT)

此種網路型態與 host-only networking 的架構很像，但是在虛擬網路上多了一台 NAT router。請參考 VMware 的官方[示意圖](https://www.vmware.com/support/ws5/doc/ws_net_configurations_nat.html)。

因為有了這台虛擬的 NAT router，guest 雖然與外界隔離，但仍然可以很方便地透過連接在 host OS 的網路連接 Internet。

這種架構也不會受到實體網路的影響，即使把網路線拔掉，host OS 和 guest OS 還是可以互通。若連接到 host OS 的 Internet 連線沒有問題，guest OS 也一樣可以連接到 Internet。
