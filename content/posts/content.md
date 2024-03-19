---
author: ["Aditiya Darmawan"]
title: "Network Infra Implementation"
date: "2024-03-19"
description: "Using MikroTik Router (RB951Ui-2HnD) Switch (CSS326-24G-2S+)"
summary: "Article showcasing implementation vlan, vlsm, firewall."
tags: ["MikroTik", "Vlan", "Firewall"]
categories: ["themes", "syntax"]
series: ["Themes Guide"]
ShowToc: true
TocOpen: true
---

### Topologi

{{< figure align=center src="../router.jpg" >}}

### Address

`172.16.0.0/16 dibagi menjadi 7 network dengan teknik VLSM sebanyak 62 alamat ditiap subnet yang nantinya akan dialokasian ke vlan`

| VLAN            | Address                  |
| --------------- | ------------------------------- |
| `10`         | 172.16.0.1/26 - 172.16.0.62/26 |
| `20`         | 172.16.0.65/26 - 172.16.0.62/26 |
| `30`         | 172.16.0.129/26 - 172.16.0.62/26 |
| `40`         | 172.16.0.193/26 - 172.16.0.62/26 |
| `50`         | 172.16.1.1/26 - 172.16.0.62/26 |
| `60`         | 172.16.1.65/26 - 172.16.0.62/26 |
| `70`         | 172.16.1.129/26 - 172.16.0.62/26 |

`Vlan Management dibuat dengan alamat berikut`

| VLAN            | Address                  |
| --------------- | ------------------------------- |
| `99`         | 99.99.99.1/29 - 99.99.99.2/29 |


### Blocked Policy

Block User yang mengunduh lebih dari 30 Mb

```bash {linenos=true}
/ip firewall filter
add action=passthrough chain=unused-hs-chain comment=\
    "place hotspot rules here" disabled=yes
add action=add-src-to-address-list address-list=client-download \
    address-list-timeout=15m chain=forward comment="Block Download 30MB" \
    connection-bytes=30000000-0 connection-rate=1M-10M disabled=yes protocol=\
    tcp src-address=172.16.0.192/26
add action=add-src-to-address-list address-list=client-download \
    address-list-timeout=15m chain=forward comment="Block Download 30MB" \
    connection-bytes=30000000-0 connection-rate=1M-100M disabled=yes \
    protocol=tcp src-address=172.16.1.0/26
add action=add-src-to-address-list address-list=client-download \
    address-list-timeout=15m chain=forward comment="Block Download 30MB" \
    connection-bytes=30000000-0 connection-rate=1M-100M disabled=yes \
    protocol=tcp src-address=172.16.0.128/26
add action=add-src-to-address-list address-list=client-download \
    address-list-timeout=15m chain=forward comment="Block Download 30MB" \
    connection-bytes=30000000-0 connection-rate=1M-100M disabled=yes \
    protocol=udp src-address=172.16.0.128/26
add action=add-src-to-address-list address-list=client-download \
    address-list-timeout=15m chain=forward comment="Block Download 30MB" \
    connection-bytes=30000000-0 connection-rate=1M-10M disabled=yes protocol=\
    udp src-address=172.16.0.192/26
add action=add-src-to-address-list address-list=client-download \
    address-list-timeout=15m chain=forward comment="Block Download 30MB" \
    connection-bytes=30000000-0 connection-rate=1M-10M disabled=yes protocol=\
    udp src-address=172.16.1.0/26
add action=drop chain=forward comment="Block Download 30MB" disabled=yes \
    protocol=tcp src-address-list=client-download
add action=drop chain=forward comment="Block Download 30MB" disabled=yes \
    protocol=udp src-address-list=client-download

```
Blokir Windows Update
```bash {linenos=true}
/ip firewall filter
add action=drop chain=prerouting comment="Blok Windows Update" protocol=tcp \
    tls-host=windowsupdate.microsoft.com
add action=drop chain=prerouting comment="Blok Windows Update" protocol=tcp \
    tls-host=download.microsoft.com
add action=drop chain=prerouting comment="Blok Windows Update" protocol=tcp \
    tls-host=test.stats.update.microsoft.com
add action=drop chain=prerouting comment="Blok Windows Update" protocol=tcp \
    tls-host=ntservicepack.microsoft.com
add action=drop chain=prerouting comment="Blok Windows Update" protocol=tcp \
    tls-host=*.download.windowsupdate.com
add action=drop chain=prerouting comment="Blok Windows Update" protocol=tcp \
    tls-host=*.update.microsoft.com
add action=drop chain=prerouting comment="Blok Windows Update" protocol=tcp \
    tls-host=download.windowsupdate.com
add action=drop chain=prerouting comment="Blok Windows Update" protocol=tcp \
    tls-host=*.windowsupdate.microsoft.com

```
Blokir Sosial Media dan Streaming Service (Tidak semua layanan karna keterbatasan cpu)

```bash {linenos=true}
/ip firewall filter
add action=drop chain=forward comment="Tiktok Block" disabled=yes protocol=\
    tcp tls-host=*.musical.ly
add action=drop chain=forward comment="Tiktok Block" content=tiktokv.com \
    disabled=yes
add action=drop chain=forward comment="Tiktok Block" content=musical.ly \
    disabled=yes
add action=drop chain=forward comment="Tiktok Block" content=tiktok disabled=\
    yes
add action=drop chain=forward comment="Twitter Block" content=twitter.com
add action=drop chain=forward comment="Twitter Block" content=.twitter.
add action=drop chain=forward comment="Youtube Block" content=googlevideo.com
add action=drop chain=forward comment="Twitter Block" content=twimg.com
add action=drop chain=forward comment="Telegram Block" content=telegram.com
add action=drop chain=forward comment="Telegram Block" content=.telegram.
add action=drop chain=forward comment="Netflix Block" content=netflix.com
add action=drop chain=forward comment="Netflix Block" content=.netflix.
add action=drop chain=forward comment="Instagram Block" content=instagram.com
add action=drop chain=forward comment="Instagram Block" content=.Instagram.
add action=drop chain=forward comment="Youtube Block" content=youtube.com
add action=drop chain=forward comment="Youtube Block" content=.youtube.
add action=drop chain=forward comment="Youtube Block" content=.googlevideo.
```
### DHCP Server

dhcp disetup untuk memudahkan pemberian alamat ip ke komputer

```bash {linenos=true}
/ip dhcp-server
add address-pool=dhcp_pool0 interface=R-TIK lease-time=12h name=R-TIK
add address-pool=dhcp_pool1 interface=R-ANM lease-time=12h name=R-ANM
add address-pool=dhcp_pool2 interface=R-BC lease-time=12h name=R-BC
add address-pool=dhcp_pool3 interface=R-Editing lease-time=12h name=R-Editing
add address-pool=dhcp_pool5 interface=R-RPL lease-time=12h name=RPL
/ip dhcp-server network
add address=10.10.10.0/24 gateway=10.10.10.1
add address=172.16.0.0/26 dns-server=94.140.14.15,94.140.15.16 gateway=\
    172.16.0.1
add address=172.16.0.64/26 dns-server=94.140.14.15,94.140.15.16 gateway=\
    172.16.0.65
add address=172.16.0.128/26 dns-server=94.140.14.15,94.140.15.16 gateway=\
    172.16.0.129
add address=172.16.0.192/26 dns-server=94.140.14.15,94.140.15.16 gateway=\
    172.16.0.193
add address=172.16.1.64/26 gateway=172.16.1.65
add address=192.168.10.0/24 gateway=192.168.10.1
```
