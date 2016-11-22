---
title: Network settings for virtualbox
date: 2016-11-21 16:49:04
tags:
  - vm
  - network
  - linux
---

- VM: CentOS 6.5
- Host: Windows7

## VM 設定

1. 網卡設定
   - 第一張設 **NAT**
   - 第二張設 **橋接介面卡**

2. 確認 VM 網卡設定 
    ```shell
    $ ifconfig
    ```
   - 確認兩張網卡都有開 `eth0` / `eth1`
   - 沒有的話，就啟動網卡
   ```shell
   $ ifconfig {eth0|eth1} {up|down}
   ```
3. VM網卡設定
    
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
   - 設定完記得重啟
   ```shell
   $ /etc/init.d/network restart
   ```
4. 檢查連線
```shell
$ ping 8.8.8.8
```

## 本機設定

1. 確定有加網路區段
    > 區域連線 > IPv4 > 進階 > IP設定

   增加和VM同網段的IP位址 `192.168.XXX.LLL`
   
2. 檢查是否連得到 VM
```shell
$ ping 192.168.XXX.VVV
```

## 其他設定

- 關閉防火牆
```shell
$ service iptables stop
```
- 設定開機時不會開起防火牆
```
$ chkconfig iptables off
```