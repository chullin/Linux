# 路由器設定 (10/1)

```
en
conf t
hostname R1
int e0/0
ip addr 12.1.1.1 255.255.255.0
no shut
int lo 1
ip addr 1.1.1.1 255.255.255.255
do show ip int brief
以相同方式不同 IP 配置第二台

R1 > ping 12.1.1.2
.!!!!
第一封包為了執行 ARP 解析，所以第一個封包會被丟棄

show ip route 在路由器裡面看路由表

ping x.x.x.x repeat 數字   ( 重複ping '數字' 次 ) 

ping x.x.x.x source y.y.y.y   ( ping 來源 y.y.y.y )

ip route 1.1.1.1 255.255.255.255 12.1.1.1  ( 將第二台機器新增去第一台的路徑 )
```

Address Resolution Protocol ( ARP 位址解析協定 )

是一個通過解析網路層位址來找尋資料鏈路層位址的網路傳輸協定


<img src="images/25.jpg" width="500px">

<img src="images/26.jpg" width="500px">

<img src="images/27.jpg" width="500px">

### 三台路由器

預設路由（Default route），是對IP數據包中的目的地址找不到存在的其他路由時，路由器所選擇的路由。目的地不在路由器的路由表里的所有數據包都會使用預設路由

IPv4預設路由是0.0.0.0/0

```
ip route 0.0.0.0 0.0.0.0 13.1.1.3 ?     觀看可添加參數，預設值為 0
no ip route 0.0.0.0 0.0.0.0 13.1.1.3    取消路由設定

do show ip static route 0.0.0.0/0       觀看到某一位址的路徑有多少條，全數列出，此路由屬於靜態路由 

debug ip icmp                           觀測封包
```

<img src="images/28.jpg" width="500px">


Cisco Express Forwarding ( CEF 思科快遞交換)
CEF可以將 route lookup 的結果保存在 CEF Table，當 router 再次收到同一個 Destination 的 Packet 的話，不用再次尋找 route table




# RIP 協定 (10/8)

R1 設定如下
```
en
conf t
hostname R1
int e0/0
ip addr 10.1.1.1 255.255.255.0
no shut
exit
router rip
network 10.1.1.1
do show run
```

R2 設定如下
```
hostname R2 {
  interface Ethernet 0/0
  ip address 10.1.1.2 255.255.255.0
  interface Ethernet 0/1
  ip address 10.1.2.1 255.255.255.0
}
router rip
  version 2
  network 10.1.2.1
```

R3 設定如下
```
hostname R3 {
  interface Ethernet 0/0
  ip address 10.1.2.2 255.255.255.0

}
router rip
  version 2
  network 10.1.2.2
```
<img src="images/29.jpg" width="500px">

<img src="images/30.jpg" width="500px">

<img src="images/31.jpg" width="500px">

<img src="images/32.jpg" width="500px">

<img src="images/33.jpg" width="500px">


# Generic Routing Encapsulation (GRE) (10/22)

GRE 可以在兩個 Physical interface 之間建立點對點 Tunnel，多用於設置 Virtual Private Network(VPN) 去保護資訊

先建立以下拓樸

<img src="images/34.png" width="500px">

<img src="images/35.jpg" width="500px">

## 在 R1 R2 之間建立 tunnel
```
R1> int tunnel 12
R1> ip address 172.16.12.1 255.255.255.0
R1> tunnel source ethernet 1/0
R1> tunnel destionation 10.0.24.2
```
```
R2> int tunnel 12
R2> ip address 172.16.12.2 255.255.255.0
R2> tunnel source ethernet 1/0
R2> tunnel destionation 10.0.14.1
```

在 R1 R3 之間建立 tunnel
```
R1> int tunnel 13
R1> ip address 172.16.13.1 255.255.255.0
R1> tunnel source ethernet 1/0
R1> tunnel destionation 10.0.34.3
```
```
R2> int tunnel 13
R2> ip address 172.16.13.2 255.255.255.0
R2> tunnel source ethernet 1/0
R2> tunnel destionation 10.0.14.1
```
確認 R1、R2、R3 是否互相連通
```
R1> ping 172.16.12.2 source 172.16.12.1
R1> ping 172.16.13.3 source 172.16.12.1

R2> ping 172.16.13.3 source 172.16.12.2
R2> traceroute 172.16.13.3 route 172.16.12.2    traceroute 為從你的電腦到互聯網另一端的主機是走的什麼路徑，會發現路徑是 R2 > R1 > R3，因為是 Hub-to-spoke Topology
```
## Routing Protocol

利用 EIGRP 將 Router 背後的網路互相發布

```
R1> router eigrp 1
R1> network 172.16.12.0 0.0.0.255    0.0.0.255 為反掩碼
R1> network 172.16.13.0 0.0.0.255
R1> network 192.168.1.0
R1> no auto-summary                  

auto summary
這個命令的作用是關閉路由協議的自動匯總功能,主要是為了解決不連續子網互相訪問的問題,在這種情況下都會關閉自動匯總,而採用手工匯總的方式通告路由 

R2> router eigrp 1
R2> network 172.16.12.0 0.0.0.255    0.0.0.255 為反掩碼
R2> network 192.168.2.0
R2> no auto-summary                  


R3> router eigrp 1
R3> network 172.16.13.0 0.0.0.255    0.0.0.255 為反掩碼
R3> network 192.168.3.0
R3> no auto-summary  
```

查看 EIGRP 是否成功交換路由表
```
show ip route eigrp 1
traceroute 172.168.12.3 source 172.16.12.2
```


# IPSec (10/29)

<img src="images/39.jpg" width="500px">

```
R1 e0/0 為 192.168.10.1 255.255.255.0
   e0/1 為 192.168.13.1 255.255.255.0

ip route 0.0.0.0 0.0.0.0 192.168.13.3

ip access-list extended VPN-traffic
permit ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
```

```
R2 e0/0 為 192.168.20.1 255.255.255.0
   e0/1 為 192.168.23.2 255.255.255.0

ip route 0.0.0.0 0.0.0.0 192.168.23.3

ip access-list extended VPN-traffic
permit ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
```

```
R3 e0/0 為 192.168.13.3 255.255.255.0
   e0/1 為 192.168.23.3 255.255.255.0
```

```
R1> crypto isakmp policy 1
R1> encryption aes
R1> hash md5
R1> authentication pre-share
R1> group 2
R1> exit
R1> crypto ipsec transform-set TS esp-3des ah-sha-hmac
R1> exit
R1> crypto isakmp key 6 ccie address 192.168.23.2

R1> crypto map CMAP 1 ipsec-isakmp
R1> set peer 192.168.23.2
R1> set transforn-set TS
R1> match address VPN-Traffic
R1> exit
R1> int e0/0
R1> crypto map CMAP
```

```
R2> crypto isakmp policy 1
R2> encryption aes
R2> hash md5
R2> authentication pre-share
R2> group 2
R2> exit
R2> crypto ipsec transform-set TS esp-3des ah-sha-hmac
R2> exit
R2> crypto isakmp key 6 ccie address 192.168.13.1

R2> crypto map CMAP 1 ipsec-isakmp
R2> set peer 192.168.13.1
R2> set transforn-set TS
R2> match address VPN-Traffic
R2> exit
R2> int e0/1
R2> crypto map CMAP
```

```
VPC4> ip 192.168.10.10 255.255.255.0 gateway 192.168.10.1

VPC5> ip 192.168.20.10 255.255.255.0 gateway 192.168.20.1
```

# GRE over IPSec vs IPSec over GRE

<img src="images/37.jpg" width="500px">

GRE Tunnel 並無加密功能，流經 Internet 的資訊變得不安全，這時候 GRE 可與 IPSec 一起應用

IPSec：數據機密性、數據完整性、身分驗證、防重放攻擊

## GRE over IPSec

<img src="images/36.jpg" width="500px">

第一個方法是 GRE over IPSec，即 IPSec 在最外層(或稱最底層)。意思是先在 R1 與 R2 之間建立 IPSec Tunnel，把裡面的 GRE Tunnel 整個進行加密，Routing Protocol 在 GRE Tunnel 裡面完成 Route 交換，最後 Data 在 GRE Tunnel 裡面傳送。從下圖所見，因整個 GRE Tunnel 被加密，所以裡面的 Routing Protocol 及 Data 都會被加密

<img src="images/40.jpg" width="500px">

```
R1 lo   為 1.1.1.1 255.255.255.0
   e1/0 為 192.168.13.1 255.255.255.0

ip route 192.168.23.0 255.255.255.0 192.168.13.3
```

```
R2 lo   為 2.2.2.2 255.255.255.0
   e1/0 為 192.168.23.2 255.255.255.0

ip route 192.168.13.0 255.255.255.0 192.168.23.2
```

```
R3 e1/0 為 192.168.13.3 255.255.255.0
   e1/1 為 192.168.23.3 255.255.255.0
```

接著確認只有 192.168.13.1 能 ping 192.168.23.2

```
R1> ping 192.168.23.25 source 192.168.23.1
!!!!!

R2> ping 2.2.2.2 source 1.1.1.1
.....
```

```
R1> ip access-list extended IPSEC_TUNNEL
R1> permit ip host 192.168.13.1 host 192.168.23.2
```

```
R2> ip access-list extended IPSEC_TUNNEL
R2> permit ip host 192.168.23.2 host 192.168.13.1
```


* [回上一頁](https://github.com/chullin/Linux/)