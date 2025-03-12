# OTUS PRO Homework 33 OSPF

## Домашняя работа 33: Статическая и динамическая маршрутизация, OSPF

### Домашнее задание:
Описание домашнего задания:

1. Подготовка рабочего места

2. Развернуть 3 виртуальные машины

3. Объединить их разными vlan
* настроить OSPF между машинами на базе Quagga;
* изобразить ассиметричный роутинг;
* сделать один из линков "дорогим", но что бы при этом роутинг был симметричным. 
---
### 1. Подготовка рабочего места:
Выполнение домашнего задания предполагает, что на компьютере установлен Vagrant+VirtualBox   
**[Как установить Vagrant на Debian 12](https://github.com/avlikh/Install_Vagrant_Debian12/blob/main/README.md)**   

Развернем Vagrant-стенд:
  - Создайте папку с проектом и зайдите в нее (например: /opt/otus/ospf):
```
mkdir -p /opt/otus/ospf ; cd /opt/otus/ospf
```
  - Клонируете проект с Github, набрав команду:
```
apt update -y && apt install git -y ; git clone https://github.com/avlikh/Otus_pro_33.git .
```
  - Запустите проект из папки, в которую склонировали проект (в нашем примере /opt/otus/ospf):
```
vagrant up
```
Результатом выполнения команды vagrant up станет созданные и настроенные 3 виртуальные машины:    
    
* **router1** 
* **router2** 
* **router3**   

---


## Выполнение задания:
### 2. Развернуть 3 виртуальные машины:    
    
В результате подготовки рабочего места (пункт 1) были созданы 3 виртуальные машины, к которым были применены необходимые настройки.    
Был выполнен playbook Ansible https://github.com/avlikh/Otus_pro_33/blob/main/provision.yml.    
Созданные виртуальные машины соединены между сосбой, согласно схеме сети (см. ниже)

---

### 3. Объединить их разными vlan:
* **3.1 настроить OSPF между машинами на базе Quagga;**
* **3.2 изобразить ассиметричный роутинг;**
* **3.3 сделать один из линков "дорогим", но что бы при этом роутинг был симметричным.**
     
    
**Для начала нарисуем схему сети:**

![image](https://github.com/user-attachments/assets/33a02588-7054-4930-bdc5-cff49e2cc41e)
    
**3.1 настроить OSPF между машинами на базе Quagga;**    
     
Проверим работу нашего стенда.    

Для этого, подключимся к **router1**.    
`vagrant ssh router1`    
`sudo -i`    
    
Проверим что служба frr запущена:

```
Systemctl status frr
```

<details>
<summary> результат выполнения команды </summary>

```
● frr.service - FRRouting
     Loaded: loaded (/lib/systemd/system/frr.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2025-03-12 11:51:13 UTC; 33min ago
       Docs: https://frrouting.readthedocs.io/en/latest/setup.html
    Process: 5732 ExecStart=/usr/lib/frr/frrinit.sh start (code=exited, status=0/SUCCESS)
   Main PID: 5750 (watchfrr)
     Status: "FRR Operational"
      Tasks: 10 (limit: 1117)
     Memory: 20.8M
     CGroup: /system.slice/frr.service
             ├─5750 /usr/lib/frr/watchfrr -d -F traditional zebra mgmtd ospfd staticd
             ├─5761 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 90000000
             ├─5766 /usr/lib/frr/mgmtd -d -F traditional -A 127.0.0.1
             ├─5768 /usr/lib/frr/ospfd -d -F traditional -A 127.0.0.1
             └─5771 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1

Mar 12 11:51:08 router1 ospfd[5768]: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
Mar 12 11:51:08 router1 frrinit.sh[5778]: line 53: % Unknown command[4]: default-information originate always[5778|ospfd] Configuration file[/etc/frr/frr.conf] processing failure: 2
Mar 12 11:51:08 router1 watchfrr[5750]: [ZJW5C-1EHNT] restart all process 5751 exited with non-zero status 2
Mar 12 11:51:13 router1 watchfrr[5750]: [QDG3Y-BY5TN] staticd state -> up : connect succeeded
Mar 12 11:51:13 router1 watchfrr[5750]: [QDG3Y-BY5TN] mgmtd state -> up : connect succeeded
Mar 12 11:51:13 router1 watchfrr[5750]: [QDG3Y-BY5TN] zebra state -> up : connect succeeded
Mar 12 11:51:13 router1 watchfrr[5750]: [QDG3Y-BY5TN] ospfd state -> up : connect succeeded
Mar 12 11:51:13 router1 watchfrr[5750]: [KWE5Q-QNGFC] all daemons up, doing startup-complete notify
Mar 12 11:51:13 router1 frrinit.sh[5732]:  * Started watchfrr
Mar 12 11:51:13 router1 systemd[1]: Started FRRouting.
```
</details>

Видим, что служба frr запущена.    
    
<details>
<summary> Проверим доступность сетей: </summary>

```
root@router1:~# ping -c2 192.168.10.1
PING 192.168.10.1 (192.168.10.1) 56(84) bytes of data.
64 bytes from 192.168.10.1: icmp_seq=1 ttl=64 time=0.017 ms
64 bytes from 192.168.10.1: icmp_seq=2 ttl=64 time=0.091 ms

--- 192.168.10.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1030ms
rtt min/avg/max/mdev = 0.017/0.054/0.091/0.037 ms
root@router1:~# ^C
root@router1:~# ^C
root@router1:~# ping -c2 192.168.20.1
PING 192.168.20.1 (192.168.20.1) 56(84) bytes of data.
64 bytes from 192.168.20.1: icmp_seq=1 ttl=63 time=2.57 ms
64 bytes from 192.168.20.1: icmp_seq=2 ttl=63 time=2.80 ms

--- 192.168.20.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1003ms
rtt min/avg/max/mdev = 2.565/2.680/2.796/0.115 ms
root@router1:~# ^C
root@router1:~# ^C
root@router1:~# ping -c2 192.168.30.1
PING 192.168.30.1 (192.168.30.1) 56(84) bytes of data.
64 bytes from 192.168.30.1: icmp_seq=1 ttl=64 time=0.610 ms
64 bytes from 192.168.30.1: icmp_seq=2 ttl=64 time=1.45 ms

--- 192.168.30.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1030ms
rtt min/avg/max/mdev = 0.610/1.032/1.454/0.422 ms
root@router1:~# ^C
root@router1:~# ^C
root@router1:~# ping -c2 10.0.10.1
PING 10.0.10.1 (10.0.10.1) 56(84) bytes of data.
64 bytes from 10.0.10.1: icmp_seq=1 ttl=64 time=0.020 ms
64 bytes from 10.0.10.1: icmp_seq=2 ttl=64 time=0.092 ms

--- 10.0.10.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1024ms
rtt min/avg/max/mdev = 0.020/0.056/0.092/0.036 ms
root@router1:~# ^C
root@router1:~# ^C
root@router1:~# ping -c2 10.0.11.1
PING 10.0.11.1 (10.0.11.1) 56(84) bytes of data.
64 bytes from 10.0.11.1: icmp_seq=1 ttl=64 time=0.530 ms
64 bytes from 10.0.11.1: icmp_seq=2 ttl=64 time=1.59 ms

--- 10.0.11.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1031ms
rtt min/avg/max/mdev = 0.530/1.060/1.591/0.530 ms
root@router1:~# ^C
root@router1:~# ^C
root@router1:~# ping -c2 10.0.12.1
PING 10.0.12.1 (10.0.12.1) 56(84) bytes of data.
64 bytes from 10.0.12.1: icmp_seq=1 ttl=64 time=0.019 ms
64 bytes from 10.0.12.1: icmp_seq=2 ttl=64 time=0.095 ms

--- 10.0.12.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1006ms
rtt min/avg/max/mdev = 0.019/0.057/0.095/0.038 ms
```
</details>

Видим, что все сети доступны.
        
    

<details>
<summary> Посмотрим маршруты до 192.168.30.0/24 сети с включенным и выключенным интерфейсом enp0s9 </summary>

```
root@router1:~# traceroute 192.168.30.1
traceroute to 192.168.30.1 (192.168.30.1), 30 hops max, 60 byte packets
 1  192.168.30.1 (192.168.30.1)  0.457 ms  0.412 ms  0.388 ms
root@router1:~# ^C
root@router1:~# ^C
root@router1:~# ifconfig enp0s9 down
root@router1:~# ^C
root@router1:~# ^C
root@router1:~# traceroute 192.168.30.1
traceroute to 192.168.30.1 (192.168.30.1), 30 hops max, 60 byte packets
 1  10.0.10.2 (10.0.10.2)  0.488 ms  0.447 ms  0.425 ms
 2  192.168.30.1 (192.168.30.1)  0.861 ms  0.976 ms  0.782 ms
```
</details>
      
Видим, что при отключении интерфейса enp0s9, маршрут перестраивается.      

<details>
<summary> Посмотрим таблицу маршрутизации роутера </summary>

```
root@router1:~# ifconfig enp0s9 up
root@router1:~# vtysh

Hello, this is FRRouting (version 10.2.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

router1# show ip route ospf
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

O   10.0.10.0/30 [110/1000] is directly connected, enp0s8, weight 1, 00:47:52
O>* 10.0.11.0/30 [110/200] via 10.0.12.2, enp0s9, weight 1, 00:00:21
O   10.0.12.0/30 [110/100] is directly connected, enp0s9, weight 1, 00:00:21
O   192.168.10.0/24 [110/100] is directly connected, enp0s10, weight 1, 00:48:22
O>* 192.168.20.0/24 [110/300] via 10.0.12.2, enp0s9, weight 1, 00:00:21
O>* 192.168.30.0/24 [110/200] via 10.0.12.2, enp0s9, weight 1, 00:00:21
```
</details>

Видим, что все маршруты построены корректно.    
    
**Повторим те же действия на хостах router2 и router3:

