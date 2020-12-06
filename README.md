## **Задание**

В Office1 в тестовой подсети появляется сервера с доп интерфесами и адресами
в internal сети testLAN:

- testClient1 - 10.10.10.254
- testClient2 - 10.10.10.254
- testServer1- 10.10.10.1
- testServer2- 10.10.10.1

Развести вланами:

testClient1 <-> testServer1
testClient2 <-> testServer2

Между centralRouter и inetRouter "пробросить" 2 линка (общая inernal сеть) и объединить их в бонд, проверить работу c отключением интерфейсов.

## ** Ход выполнения задания**

Топология сети:

![alt text](./vlan-bonding.png)

Стартуем виртуалки `vagrant up`.

## Проверим сначала vlan'ы***

**Зайдем на testServer1**:
```
[root@testServer1 ~]# ip --brief link show
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             52:54:00:8a:fe:e6 <BROADCAST,MULTICAST,UP,LOWER_UP>
eth1             UP             08:00:27:0f:8a:04 <BROADCAST,MULTICAST,UP,LOWER_UP>
eth2             UP             08:00:27:65:69:1e <BROADCAST,MULTICAST,UP,LOWER_UP>
eth1.10@eth1     UP             08:00:27:0f:8a:04 <BROADCAST,MULTICAST,UP,LOWER_UP>
[root@testServer1 ~]# ip --brief addr show
lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             10.0.2.15/24 fe80::5054:ff:fe8a:fee6/64
eth1             UP             192.168.2.2/24 fe80::a00:27ff:fe0f:8a04/64
eth2             UP             10.11.11.111/24 fe80::a00:27ff:fe65:691e/64
eth1.10@eth1     UP             10.10.10.1/24 fe80::a00:27ff:fe0f:8a04/64
[root@testServer1 ~]# ping 10.10.10.254
PING 10.10.10.254 (10.10.10.254) 56(84) bytes of data.
64 bytes from 10.10.10.254: icmp_seq=1 ttl=64 time=0.385 ms
64 bytes from 10.10.10.254: icmp_seq=2 ttl=64 time=0.906 ms
64 bytes from 10.10.10.254: icmp_seq=4 ttl=64 time=1.07 ms
64 bytes from 10.10.10.254: icmp_seq=5 ttl=64 time=1.08 ms
```

**Зайдем на testClient1**:
```
[root@testClient1 ~]# ip --brief link show
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             52:54:00:8a:fe:e6 <BROADCAST,MULTICAST,UP,LOWER_UP>
eth1             UP             08:00:27:b2:be:13 <BROADCAST,MULTICAST,UP,LOWER_UP>
eth2             UP             08:00:27:b0:aa:9b <BROADCAST,MULTICAST,UP,LOWER_UP>
eth1.10@eth1     UP             08:00:27:b2:be:13 <BROADCAST,MULTICAST,UP,LOWER_UP>
[root@testClient1 ~]# ip --brief addr show
lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             10.0.2.15/24 fe80::5054:ff:fe8a:fee6/64
eth1             UP             192.168.2.3/24 fe80::a00:27ff:feb2:be13/64
eth2             UP             10.11.11.121/24 fe80::a00:27ff:feb0:aa9b/64
eth1.10@eth1     UP             10.10.10.254/24 fe80::a00:27ff:feb2:be13/64
[root@testClient1 ~]# ping 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.824 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.747 ms
64 bytes from 10.10.10.1: icmp_seq=4 ttl=64 time=0.919 ms
^C

```

**Теперь на testServer2**:
```
[root@testServer2 ~]# ip --brief link show
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             52:54:00:8a:fe:e6 <BROADCAST,MULTICAST,UP,LOWER_UP>
eth1             UP             08:00:27:b2:03:de <BROADCAST,MULTICAST,UP,LOWER_UP>
eth2             UP             08:00:27:c2:7b:f3 <BROADCAST,MULTICAST,UP,LOWER_UP>
eth1.20@eth1     UP             08:00:27:b2:03:de <BROADCAST,MULTICAST,UP,LOWER_UP>
[root@testServer2 ~]# ip --brief addr show
lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             10.0.2.15/24 fe80::5054:ff:fe8a:fee6/64
eth1             UP             192.168.2.4/24 fe80::a00:27ff:feb2:3de/64
eth2             UP             10.11.11.112/24 fe80::a00:27ff:fec2:7bf3/64
eth1.20@eth1     UP             10.10.10.1/24 fe80::a00:27ff:feb2:3de/64
[root@testServer2 ~]# ping 10.10.10.254
PING 10.10.10.254 (10.10.10.254) 56(84) bytes of data.
64 bytes from 10.10.10.254: icmp_seq=1 ttl=64 time=0.619 ms
64 bytes from 10.10.10.254: icmp_seq=2 ttl=64 time=0.921 ms
```

**аналогично проверим ip связность и состояние интерфейсов на testClient2**:
```
[root@testClient2 ~]# ip --brief link show
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             52:54:00:8a:fe:e6 <BROADCAST,MULTICAST,UP,LOWER_UP>
eth1             UP             08:00:27:11:eb:78 <BROADCAST,MULTICAST,UP,LOWER_UP>
eth2             UP             08:00:27:85:76:bb <BROADCAST,MULTICAST,UP,LOWER_UP>
eth1.20@eth1     UP             08:00:27:11:eb:78 <BROADCAST,MULTICAST,UP,LOWER_UP>
[root@testClient2 ~]# ip --brief addr show
lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             10.0.2.15/24 fe80::5054:ff:fe8a:fee6/64
eth1             UP             192.168.2.5/24 fe80::a00:27ff:fe11:eb78/64
eth2             UP             10.11.11.122/24 fe80::a00:27ff:fe85:76bb/64
eth1.20@eth1     UP             10.10.10.254/24 fe80::a00:27ff:fe11:eb78/64
[root@testClient2 ~]# ping 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.744 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.820 ms
```

## ***Проверим bonding между роутерами и отказоусточивоть линка***

**inetRouter**:
```
[root@inetRouter ~]# cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup) (fail_over_mac active)
Primary Slave: None
Currently Active Slave: eth1
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth1
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:61:06:f6
Slave queue ID: 0

Slave Interface: eth2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:5a:db:b8
Slave queue ID: 0

[root@inetRouter ~]# cat /etc/sysconfig/network-scripts/ifcfg-bond0
DEVICE=bond0
NAME=bond0
TYPE=Bond
BONDING_MASTER=yes
IPADDR=10.1.1.1
NETMASK=255.255.255.252
ONBOOT=yes
USERCTL=no
BOOTPROTO=static
BONDING_OPTS="mode=1 miimon=100 fail_over_mac=1"
ZONE=internal

```
Основные параметра, которые использованы в интерфейсе bond0

mode=1  - определяет политику поведения объединенных интерфейсов. 1 - это политика активный-резервный.

miimon=100 - Устанавливает периодичность MII мониторинга в миллисекундах. Фактически монитринг будет осуществляться 10 раз в секунду (1с = 1000 мс).

fail_over_mac=1 - Определяет как будут прописываться MAC адреса на объединенных интерфейсах в режиме active-backup при переключении интерфесов.


**Аналогично на centralRouter**:
```
root@centralRouter ~]# cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup) (fail_over_mac active)
Primary Slave: None
Currently Active Slave: eth1
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth1
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:1c:c7:db
Slave queue ID: 0

Slave Interface: eth2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:02:17:5e
Slave queue ID: 0


[root@centralRouter ~]# cat /etc/sysconfig/network-scripts/ifcfg-bond0
DEVICE=bond0
NAME=bond0
TYPE=Bond
BONDING_MASTER=yes
IPADDR=10.1.1.2
NETMASK=255.255.255.252
ONBOOT=yes
USERCTL=no
BOOTPROTO=static
BONDING_OPTS="mode=1 miimon=100 fail_over_mac=1"
GATEWAY=10.1.1.1

```

Видим, что активным интерфесом на обоих роутерах выбран `eth1`. Запустим пинги фоново в текстовый файлик и выключим `eth1` на **inetRouter**:
```
[root@inetRouter ~]# ping 10.1.1.2 >ping_chek.txt &
[1] 24755
[root@inetRouter ~]# ip link set eth1 down
[root@inetRouter ~]# cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup) (fail_over_mac active)
Primary Slave: None
Currently Active Slave: eth2
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth1
MII Status: down
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 1
Permanent HW addr: 08:00:27:97:77:82
Slave queue ID: 0

Slave Interface: eth2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:3a:d0:1c
Slave queue ID: 0


[root@inetRouter ~]# cat ping_chek.txt
PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=0.496 ms
64 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=1.00 ms
64 bytes from 10.1.1.2: icmp_seq=3 ttl=64 time=1.11 ms
64 bytes from 10.1.1.2: icmp_seq=4 ttl=64 time=1.17 ms
64 bytes from 10.1.1.2: icmp_seq=5 ttl=64 time=1.18 ms
64 bytes from 10.1.1.2: icmp_seq=6 ttl=64 time=0.519 ms
...
--- 10.1.1.2 ping statistics ---
46 packets transmitted, 46 received, 0% packet loss, time 45082ms
rtt min/avg/max/mdev = 0.390/1.062/1.925/0.235 ms

```


видно, что после выключения `eth1`, активным интерфейсом стал `eth2`, пинги при этом не прекратились и потерь пакетов не было.
