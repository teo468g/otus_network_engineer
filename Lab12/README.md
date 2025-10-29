Конфигурационные файлы  
[R12](../Lab12/Configs/R12.txt)  
[R13](../Lab12/Configs/R13.txt)  
[R14](../Lab12/Configs/R14.txt)  
[R15](../Lab12/Configs/R15.txt)  
[R18](../Lab12/Configs/R18.txt)  
[R19](../Lab12/Configs/R19.txt)  
[R20](../Lab12/Configs/R20.txt)   
[R28](../Lab12/Configs/R28.txt) 


1. Настроите NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001. 

R14#sh ip nat tr  
Pro Inside global      Inside local       Outside local      Outside global  
icmp 192.168.1.0:0     10.0.0.14:0        192.168.1.1:0      192.168.1.1:0  

R15>sh ip nat translations  
Pro Inside global      Inside local       Outside local      Outside global  
icmp 192.168.1.2:0     10.0.0.15:0        192.168.1.3:0      192.168.1.3:0  

2. Настроите NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.  

R18#sh ip nat tr  
Pro Inside global      Inside local       Outside local      Outside global  
icmp 172.17.0.17:1     10.0.0.18:1        192.168.1.5:1      192.168.1.5:1  
--- 172.17.0.17        10.0.0.18          ---                ---  

3. Настроите статический NAT для R20.  

R20# sh ip nat tr  
Pro Inside global      Inside local       Outside local      Outside global  
--- 10.0.0.15          172.16.80.21       ---                ---  

4. Настроите NAT так, чтобы R19 был доступен с любого узла для удаленного управления.  
 
 R19  
interface Loopback0  
 ip address 10.0.0.19 255.255.255.255  
 ip nat inside  
 ip virtual-reassembly in  
 ip ospf network point-to-point  
 ip ospf 1 area 101  


interface Ethernet0/0  
 description to R14  
 ip address 172.16.6.19 255.255.255.0  
 ip nat outside  
 ip virtual-reassembly in  
 ip ospf 1 area 101  

 access-list 100 permit tcp any host 10.0.0.19 eq 22   
ip nat inside source list 100 interface Ethernet0/0 overload  

5*. Настроите статический NAT(PAT) для офиса Чокурдах. 

R28#sh ip nat tr  
Pro Inside global      Inside local       Outside local      Outside global  
--- 192.168.1.15       172.18.1.29        ---                ---  


5. Настроите для IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP.  

R12:  
ip dhcp excluded-address 172.16.1.12    

ip dhcp pool VPC1    
 network 172.16.1.0 255.255.255.0    
 default-router 172.16.1.12    

VPCS> dhcp -x  
VPCS> dhcp -r
DDORA IP 172.16.1.1/24 GW 172.16.1.12 

R13:
ip dhcp excluded-address 172.16.22.13  
ip dhcp excluded-address 172.16.22.1 172.16.22.6  
ip dhcp excluded-address 172.16.22.8 172.16.22.254  

ip dhcp pool VPC7  
 network 172.16.22.0 255.255.255.0  
 default-router 172.16.22.13  

VPCS> dhcp -x  
VPCS> dhcp -r  
DDORA IP 172.16.22.7/24 GW 172.16.22.13  

6. Настроите NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13. 

R12: 
ntp master 1  
ntp update-calendar  

interface Ethernet0/2  
 description to R14  
 ip address 172.16.4.12 255.255.255.0  
 ntp broadcast  

R14:  
interface Ethernet0/0  
 description to R12  
 ip address 172.16.4.14 255.255.255.0  
 ntp broadcast client  

 R14#sh ntp associations

  address         ref clock       st   when   poll reach  delay  offset   disp  
* 172.16.4.12     .LOCL.           1     54     64   377 -1.000  -0.500  2.898  
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured  


Можно указать адрес ntp сервера командой ntp server 172.16.4.12  
SW3# sh ntp associations

  address         ref clock       st   when   poll reach  delay  offset   disp  
 ~172.16.4.12     .INIT.          16      -    512     0  0.000   0.000 15937  

   
 



















