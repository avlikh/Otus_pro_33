# OTUS PRO Homework 33 OSPF

## Домашняя работа 33: Статическая и динамическая маршрутизация, OSPF

### Домашнее задание:
Описание домашнего задания:

1. Скачать и развернуть Vagrant-стенд https://github.com/erlong15/otus-linux/tree/network   
2. Построить следующую сетевую архитектуру:    
    
Сеть office1
* 192.168.2.0/26      # dev   
* 192.168.2.64/26     # test servers   
* 192.168.2.128/26    # managers   
* 192.168.2.192/26    # office hardware   
   
Сеть office2
* 192.168.1.0/25      # dev   
* 192.168.1.128/26    # test servers   
* 192.168.1.192/26    # office hardware   
   
Сеть central
* 192.168.0.0/28      # directors   
* 192.168.0.32/28     # office hardware   
* 192.168.0.64/26     # wifi   

![image](https://github.com/user-attachments/assets/1d7c7def-5383-4d01-ab23-2121df716a08)
   
Итого должны получиться следующие сервера:
*	inetRouter
*	centralRouter
*	office1Router
*	office2Router
*	centralServer
*	office1Server
*	office2Server   
   
**Задание состоит из 2-х частей: теоретической и практической.**    
В **теоретической части** требуется:   
*	Найти свободные подсети
*	Посчитать количество узлов в каждой подсети, включая свободные
*	Указать Broadcast-адрес для каждой подсети
*	Проверить, нет ли ошибок при разбиении   
   
В **практической части** требуется:    
*	Соединить офисы в сеть согласно логической схеме и настроить роутинг
*	Интернет-трафик со всех серверов должен ходить через inetRouter
*	Все сервера должны видеть друг друга (должен проходить ping)
*	У всех новых серверов отключить дефолт на NAT (eth0), который vagrant поднимает для связи
*	Добавить дополнительные сетевые интерфейсы, если потребуется    
    
Рекомендуется использовать Vagrant + Ansible для настройки данной схемы.
   
---
## Выполнение задания:
### 1. Теоретическая часть:

Сформируем таблицу топологии нашей сети:   

<table>
    <tr>
        <td align=center>Name</td>
        <td align=center>Network</td>
        <td align=center>Netmask</td>
        <td align=center>N</td>
        <td align=center>Hostmin</td>
        <td align=center>Hostmax</td>
        <td align=center>Broadcast </td>
    </tr>
    <tr>
        <td colspan="7" align=center>Central Network </td>
    </tr>
    <tr>
        <td align=center>Directors</td>
        <td align=center>192.168.0.0/28</td>
        <td align=center>255.255.255.240</td>
        <td align=center>14</td>
        <td align=center>192.168.0.1</td>
        <td align=center>192.168.0.14</td>
        <td align=center>192.168.0.15 </td>
    </tr>
    <tr>
        <td align=center>Office hardware</td>
        <td align=center>192.168.0.32/28</td>
        <td align=center>255.255.255.240</td>
        <td align=center>14</td>
        <td align=center>192.168.0.33</td>
        <td align=center>192.168.0.46</td>
        <td align=center>192.168.0.47 </td>
    </tr>
    <tr>
        <td align=center>Wifi(mgt network)</td>
        <td align=center>192.168.0.64/26</td>
        <td align=center>255.255.255.192</td>
        <td align=center>62</td>
        <td align=center>192.168.0.65</td>
        <td align=center>192.168.0.126</td>
        <td align=center>192.168.0.127 </td>
    </tr>
    <tr>
        <td colspan="7" align=center>Office 1 network </td>
    </tr>
    <tr>
        <td align=center>Dev</td>
        <td align=center>192.168.2.0/26</td>
        <td align=center>255.255.255.192</td>
        <td align=center>62</td>
        <td align=center>192.168.2.1</td>
        <td align=center>192.168.2.62</td>
        <td align=center>192.168.2.63 </td>
    </tr>
    <tr>
        <td align=center>Test</td>
        <td align=center>192.168.2.64/26</td>
        <td align=center>255.255.255.192</td>
        <td align=center>62</td>
        <td align=center>192.168.2.65</td>
        <td align=center>192.168.2.126</td>
        <td align=center>192.168.2.127 </td>
    </tr>
    <tr>
        <td align=center>Managers</td>
        <td align=center>192.168.2.128/26</td>
        <td align=center>255.255.255.192</td>
        <td align=center>62</td>
        <td align=center>192.168.2.129</td>
        <td align=center>192.168.2.190</td>
        <td align=center>192.168.2.191</td>
    </tr>
    <tr>
        <td align=center>Office hardware</td>
        <td align=center>192.168.2.192/26</td>
        <td align=center>255.255.255.192</td>
        <td align=center>62</td>
        <td align=center>192.168.2.193</td>
        <td align=center>192.168.2.254</td>
        <td align=center>192.168.2.255</td>
    </tr>
    <tr>
        <td colspan="7" align=center>Office 2 network</td>
    </tr>
    <tr>
        <td align=center>Dev</td>
        <td align=center>192.168.1.0/25</td>
        <td align=center>255.255.255.128</td>
        <td align=center>126</td>
        <td align=center>192.168.1.1</td>
        <td align=center>192.168.1.126</td>
        <td align=center>192.168.1.127</td>
    </tr>
    <tr>
        <td align=center>Test</td>
        <td align=center>192.168.1.128/26</td>
        <td align=center>255.255.255.192</td>
        <td align=center>62</td>
        <td align=center>192.168.1.129</td>
        <td align=center>192.168.1.190</td>
        <td align=center>192.168.1.191</td>
    </tr>
    <tr>
        <td align=center>Office</td>
        <td align=center>192.168.1.192/26</td>
        <td align=center>255.255.255.192</td>
        <td align=center>62</td>
        <td align=center>192.168.1.193</td>
        <td align=center>192.168.1.254</td>
        <td align=center>192.168.1.255</td>
    </tr>
    <tr>
        <td colspan="7" align=center>InetRouter — CentralRouter network</td>
    </tr>
    <tr>
        <td align=center>Inet — central</td>
        <td align=center>192.168.255.0/30</td>
        <td align=center>255.255.255.252</td>
        <td align=center>2</td>
        <td align=center>192.168.255.1</td>
        <td align=center>192.168.255.2</td>
        <td align=center>192.168.255.3</td>
    </tr>
</table>

После создания таблицы топологии, мы видим, что ошибок в задании нет, также мы сразу видим следующие свободные сети:   

*	192.168.0.16/28 
*	192.168.0.48/28
*	192.168.0.128/25
*	192.168.255.64/26
*	192.168.255.32/27
*	192.168.255.16/28
*	192.168.255.8/29  
*	192.168.255.4/30 
    
На этом теоретическая часть заканчивается. Можем приступать к выполнению практической части.

---
### 2. Практическая часть:

Выполнение домашнего задания предполагает, что на компьютере установлен Vagrant+VirtualBox   
**[Как установить Vagrant на Debian 12](https://github.com/avlikh/Install_Vagrant_Debian12/blob/main/README.md)**   

Развернем Vagrant-стенд:
  - Создайте папку с проектом и зайдите в нее (например: /opt/otus/network-architecture):
```
mkdir -p /opt/otus/network-architecture ; cd /opt/otus/network-architecture
```
  - Клонируете проект с Github, набрав команду:
```
apt update -y && apt install git -y ; git clone https://github.com/avlikh/Otus_pro_28.git .
```
  - Запустите проект из папки, в которую склонировали проект (в нашем примере /opt/otus/network-architecture):
```
vagrant up
```
* Примечание: Так и не получилось настроить выполнение Ansible playbook из Vagrantfile. (Все попытки сделать выполнение Ansible-playbook после создания виртуальных машин не привели к положительному результату).

По этой причине запустим Ansible playbook вручную:    
```
cd ansible
```
```
ansible-playbook provision.yml
```
В результате будет развернуто и настроено 7 виртуальных машин на Ubuntu 22
    
*	inetRouter
*	centralRouter
*	office1Router
*	office2Router
*	centralServer
*	office1Server
*	office2Server 

**Схема сети:**     
     
![image](https://github.com/user-attachments/assets/46d77015-7ae2-4834-9596-7fb2caef1fb4)
    
    

*	**Соединить офисы в сеть согласно логической схеме и настроить роутинг:**
    
Выполнено средствами Vagrant и Ansible (содержимое Vagrantfile и Ansible содержится в файлах данного проекта на Github https://github.com/avlikh/Otus_pro_28)     
     
*	**Интернет-трафик со всех серверов должен ходить через inetRouter**

Проверим что интернет-траффик ходит на всех серверах, для этого выполним taceroute хоста: 1.1.1.1     
     
<details>
<summary> centralServer </summary>

```
root@centralServer:~# traceroute 1.1.1.1
traceroute to 1.1.1.1 (1.1.1.1), 30 hops max, 60 byte packets
 1  _gateway (192.168.0.1)  0.458 ms  0.316 ms  0.284 ms
 2  192.168.255.1 (192.168.255.1)  0.801 ms  0.764 ms  0.735 ms
 3  10.0.2.2 (10.0.2.2)  0.987 ms  0.964 ms  0.881 ms
 4  10.68.0.1 (10.68.0.1)  1.274 ms  1.253 ms  1.231 ms
 5  79.99.20.145 (79.99.20.145)  2.431 ms  2.405 ms  2.504 ms
 6  100.105.105.201 (100.105.105.201)  2.471 ms  2.372 ms 100.105.105.197 (100.105.105.197)  2.449 ms
 7  * * 100.105.97.97 (100.105.97.97)  8.954 ms
 8  * * 176.99.136.121.inetcom.ru (176.99.136.121)  9.314 ms
 9  172.68.8.51 (172.68.8.51)  4.306 ms * 172.68.8.53 (172.68.8.53)  4.140 ms
10  one.one.one.one (1.1.1.1)  4.098 ms *  3.934 ms
```
</details>

<details>
<summary> office1Server </summary>

```
root@office1Server:~# traceroute 1.1.1.1
traceroute to 1.1.1.1 (1.1.1.1), 30 hops max, 60 byte packets
 1  _gateway (192.168.2.129)  2.375 ms  2.250 ms  2.724 ms
 2  192.168.255.9 (192.168.255.9)  4.824 ms  5.356 ms  5.316 ms
 3  192.168.255.1 (192.168.255.1)  5.273 ms  5.232 ms  5.183 ms
 4  10.0.2.2 (10.0.2.2)  5.311 ms  5.251 ms  1.971 ms
 5  10.68.0.1 (10.68.0.1)  2.032 ms  1.786 ms  1.772 ms
 6  79.99.20.145 (79.99.20.145)  7.679 ms  5.030 ms  4.509 ms
 7  100.105.105.201 (100.105.105.201)  4.399 ms  4.159 ms  5.776 ms
 8  100.105.97.97 (100.105.97.97)  6.148 ms  5.986 ms  5.692 ms
 9  * 91.203.28.243 (91.203.28.243)  49.939 ms  49.823 ms
10  * * *
11  one.one.one.one (1.1.1.1)  7.388 ms * *
```
</details>

<details>
<summary> office2Server </summary>

```
root@office2Server:~# traceroute 1.1.1.1
traceroute to 1.1.1.1 (1.1.1.1), 30 hops max, 60 byte packets
 1  _gateway (192.168.1.1)  0.654 ms  0.759 ms  0.680 ms
 2  192.168.255.5 (192.168.255.5)  1.381 ms  1.324 ms  1.638 ms
 3  192.168.255.1 (192.168.255.1)  2.180 ms  2.131 ms  2.422 ms
 4  10.0.2.2 (10.0.2.2)  3.025 ms  3.159 ms  3.271 ms
 5  10.68.0.1 (10.68.0.1)  3.743 ms  3.420 ms  3.121 ms
 6  79.99.20.145 (79.99.20.145)  3.401 ms  4.954 ms  4.667 ms
 7  100.105.105.197 (100.105.105.197)  4.644 ms 100.105.105.201 (100.105.105.201)  4.541 ms  4.298 ms
 8  100.105.97.97 (100.105.97.97)  4.881 ms  5.083 ms 100.105.97.110 (100.105.97.110)  47.832 ms
 9  * * *
10  * * *
11  * * one.one.one.one (1.1.1.1)  6.303 ms
```
</details>

Видим, что с каждого из 3 серверов выход в Интернет идет через **inetRouter** (ip: **192.168.255.1**)    
    
*	**Все сервера должны видеть друг друга (должен проходить ping)**    
    
<details>
<summary>пинганем с centralServer хосты: office1Server (192.168.2.130) и office2Server (192.168.1.2) </summary>

```
root@centralServer:~# ping -c2 192.168.2.130
PING 192.168.2.130 (192.168.2.130) 56(84) bytes of data.
64 bytes from 192.168.2.130: icmp_seq=1 ttl=62 time=3.80 ms
64 bytes from 192.168.2.130: icmp_seq=2 ttl=62 time=3.83 ms

--- 192.168.2.130 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 3.802/3.817/3.832/0.015 ms

root@centralServer:~# ping -c2 192.168.1.2
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=62 time=1.54 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=62 time=4.13 ms

--- 192.168.1.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1003ms
rtt min/avg/max/mdev = 1.538/2.835/4.133/1.297 ms
```
</details>

<details>
<summary>пинганем с office1Server хосты: centralServer (192.168.0.2) и office2Server (192.168.1.2) </summary>

```
root@office1Server:~# ping -c2 192.168.0.2
PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data.
64 bytes from 192.168.0.2: icmp_seq=1 ttl=62 time=3.71 ms
64 bytes from 192.168.0.2: icmp_seq=2 ttl=62 time=4.12 ms

--- 192.168.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 3.706/3.910/4.115/0.204 ms

root@office1Server:~# ping -c2 192.168.1.2
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=61 time=2.16 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=61 time=2.76 ms

--- 192.168.1.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 2.159/2.457/2.756/0.298 ms
```
</details>

<details>
<summary>пинганем с office2Server хосты: centralServer (192.168.0.2) и office1Server (192.168.2.130) </summary>

```
root@office2Server:~# ping -c2 192.168.0.2
PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data.
64 bytes from 192.168.0.2: icmp_seq=1 ttl=62 time=3.69 ms
64 bytes from 192.168.0.2: icmp_seq=2 ttl=62 time=3.91 ms

--- 192.168.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 3.685/3.796/3.908/0.111 ms

root@office2Server:~# ping -c2 192.168.2.130
PING 192.168.2.130 (192.168.2.130) 56(84) bytes of data.
64 bytes from 192.168.2.130: icmp_seq=1 ttl=61 time=1.87 ms
64 bytes from 192.168.2.130: icmp_seq=2 ttl=61 time=5.71 ms

--- 192.168.2.130 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 1.867/3.787/5.708/1.920 ms
```
</details>
    
    
* **У всех новых серверов отключить дефолт на NAT (eth0), который vagrant поднимает для связи**    
    
<details>
<summary>Выполнено в рамках Ansible Playbook: ansible/provision.yml</summary>

```
# отключаем маршрут по умолчанию
  - name: disable default route
    template: 
      src: 00-installer-config.yaml
      dest: /etc/netplan/00-installer-config.yaml
      owner: root
      group: root
      mode: 0644
    when: (ansible_hostname != "inetRouter")
```
</details>    
    
* **Добавить дополнительные сетевые интерфейсы, если потребуется**
    
Были добавлены сети для связи между **centralRouter** с **office1Router** и **office2Router**: 
* 192.168.255.4/30
* 192.168.255.8/30    