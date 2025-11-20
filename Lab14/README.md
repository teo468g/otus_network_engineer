## IPSec over DmVPN  

### GRE поверх IPSec между офисами Москва и С.-Петербург.  
## Настройка IPSec IKEv1 Route-based

R15#sh run | sec cryp  
no service password-encryption  
crypto keyring OTUS  
  pre-shared-key address 192.168.1.4 key OTUS  
crypto isakmp policy 10  
 encr aes 256  
 hash sha256  
 authentication pre-share  
 group 2  
crypto isakmp profile R15  
   keyring OTUS  
   match identity address 192.168.1.4 255.255.255.255  
   local-address 192.168.1.2  
crypto ipsec transform-set IPSEC-TS-1 esp-aes 256 esp-sha256-hmac  
 mode tunnel  
crypto ipsec profile R15  
 set transform-set IPSEC-TS-1  
 set pfs group2  
 set isakmp-profile R15 

 R15#sh run | sec Tunnel0  
interface Tunnel0  
 ip address 192.168.2.15 255.255.255.0  
 tunnel source 192.168.1.2  
 tunnel mode ipsec ipv4  
 tunnel destination 192.168.1.4  
 tunnel key 111  
 tunnel protection ipsec profile R15  

 R18#sh run | sec crypt  
no service password-encryption  
crypto keyring OTUS  
  pre-shared-key address 192.168.1.2 key OTUS  
crypto isakmp policy 10  
 encr aes 256  
 hash sha256  
 authentication pre-share  
 group 2  
crypto isakmp profile R18  
   keyring OTUS  
   match identity address 192.168.1.2 255.255.255.255  
   local-address 192.168.1.4  
crypto ipsec transform-set IPSEC-TS-1 esp-aes 256 esp-sha256-hmac  
 mode tunnel  
crypto ipsec profile R18  
 set transform-set IPSEC-TS-1  
 set pfs group2  
 set isakmp-profile R18   

 R18#sh run | sec Tunnel0  
interface Tunnel0  
 ip address 192.168.2.18 255.255.255.0  
 tunnel source 192.168.1.4  
 tunnel mode ipsec ipv4  
 tunnel destination 192.168.1.2  
 tunnel key 111  
 tunnel protection ipsec profile R18  

 Туннель построен:  
 R18#ping 192.168.2.15  
Type escape sequence to abort.  
Sending 5, 100-byte ICMP Echos to 192.168.2.15, timeout is 2 seconds:  
!!!!!  
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/6/8 ms  





 
### DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги.  

### Аутентификация по сертификатам, СА сервер на DMVPN хабе R15  

ip http server 
hostname CA  
ip host CA 192.168.1.2

Создать сервер:  
crypto pki server CA    

Создать пару ключей:  
crypto key generate rsa label CA modulus 2048  

Запросить сертификат CA:  
crypto pki authenticate CA  

Запросить сертификат для маршрутизатора:  
crypto pki enroll CA  

Разрешить запрос клиента:  
crypto pki server CA grant  


Настройка trustpoint для клиентов:  
crypto pki trustpoint CA  
 revocation-check crl  
 rsakeypair CA  
  
Trustpoint для самого сервера:   
crypto pki trustpoint myCertificate  
 enrollment url http://CA:80  
 serial-number  
 ip-address 192.168.1.2  
 revocation-check crl  

 Просмотр выданных сертификатов: 
CA#show crypto pki certificates  
Certificate  
  Status: Available  
  Certificate Serial Number (hex): 07  
  Certificate Usage: General Purpose  
  Issuer:  
    cn=CA  
  Subject:  
    Name: CA.otus.ru  
    IP Address: 192.168.1.2  
    Serial Number: 67109104  
    serialNumber=67109104+hostname=CA.otus.ru+ipaddress=192.168.1.2  
  Validity Date:  
    start date: 14:17:16 UTC Nov 20 2025  
    end   date: 14:17:16 UTC Nov 20 2026  
  Associated Trustpoints: myCertificate  
  Storage: nvram:CA#7.cer  

CA Certificate  
  Status: Available  
  Certificate Serial Number (hex): 01  
  Certificate Usage: Signature  
  Issuer:  
    cn=CA  
  Subject:  
    cn=CA  
  Validity Date:  
    start date: 16:34:57 UTC Nov 16 2025  
    end   date: 16:34:57 UTC Nov 15 2028  
  Associated Trustpoints: myCertificate CA  
  Storage: nvram:CA#1CA.cer  


 Настройка ipsec:  
 crypto isakmp policy 10  
 encr aes  
 hash sha256  
 group 16  
 lifetime 3600  
 crypto ipsec transform-set VTI esp-aes esp-sha-hmac  
 mode transport  
 crypto ipsec profile VTI  
 set transform-set VTI  

 Назначаем ipsec profile VTI на интерфейс Tunnel1:  
 tunnel protection ipsec profile VTI   
 

### R27

crypto pki trustpoint CA  
 enrollment url http://CA:80  
 serial-number  
 ip-address 192.168.1.8  
 revocation-check crl 
 
crypto isakmp policy 10  
 encr aes  
 hash sha256  
 group 16  
 lifetime 3600  
crypto ipsec transform-set VTI esp-aes esp-sha-hmac  
 mode transport  
crypto ipsec profile VTI  
 set transform-set VTI  

interface Tunnel1  
 ip address 192.168.3.27 255.255.255.0  
 no ip redirects  
 ip nhrp map multicast 192.168.1.2  
 ip nhrp map 192.168.3.15 192.168.1.2  
 ip nhrp network-id 1  
 ip nhrp nhs 192.168.3.15  
 ip nhrp registration no-unique  
 tunnel source Ethernet0/0  
 tunnel mode gre multipoint  
 tunnel key 999  
 tunnel protection ipsec profile VTI   


 ### R28

 crypto pki trustpoint CA  
 enrollment url http://CA:80  
 serial-number  
 ip-address 192.168.1.17  
 revocation-check crl  


 crypto isakmp policy 10  
 encr aes  
 hash sha256  
 group 16  
 lifetime 3600  
crypto ipsec transform-set VTI esp-aes esp-sha-hmac  
 mode transport  
crypto ipsec profile VTI  
 set transform-set VTI  


 R28#sh run | sec Tunnel1  
interface Tunnel1  
 ip address 192.168.3.28 255.255.255.0  
 no ip redirects  
 ip nhrp map multicast 192.168.1.2  
 ip nhrp map 192.168.3.15 192.168.1.2  
 ip nhrp network-id 1  
 ip nhrp nhs 192.168.3.15  
 ip nhrp registration no-unique  
 tunnel source Ethernet0/0  
 tunnel mode gre multipoint  
 tunnel key 999  
 tunnel protection ipsec profile VTI  


Проверяем доступность туннельных адресов:
CA#ping 192.168.3.27  
Type escape sequence to abort.  
Sending 5, 100-byte ICMP Echos to 192.168.3.27, timeout is 2 seconds:  
!!!!!  
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/9/10 ms  
CA#ping 192.168.3.28  
Type escape sequence to abort.  
Sending 5, 100-byte ICMP Echos to 192.168.3.28, timeout is 2 seconds:  
!!!!!  
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/8/10 ms  

Проверяем шифрование:  
CA#show crypto ipsec sa  

interface: Tunnel1  
    Crypto map tag: Tunnel1-head-0, local addr 192.168.1.2  

   protected vrf: (none)  
   local  ident (addr/mask/prot/port): (192.168.1.2/255.255.255.255/47/0)  
   remote ident (addr/mask/prot/port): (192.168.1.17/255.255.255.255/47/0)  
   current_peer 192.168.1.17 port 500  
     PERMIT, flags={origin_is_acl,}  
    #pkts encaps: 28, #pkts encrypt: 28, #pkts digest: 28  
    #pkts decaps: 19, #pkts decrypt: 19, #pkts verify: 19  
    #pkts compressed: 0, #pkts decompressed: 0  
    #pkts not compressed: 0, #pkts compr. failed: 0  
    #pkts not decompressed: 0, #pkts decompress failed: 0  
    #send errors 0, #recv errors 0  

     local crypto endpt.: 192.168.1.2, remote crypto endpt.: 192.168.1.17  
     plaintext mtu 1458, path mtu 1500, ip mtu 1500, ip mtu idb (none)  
     current outbound spi: 0x61DA50AA(1641697450)  
     PFS (Y/N): N, DH group: none  

     inbound esp sas:
      spi: 0xBA255B82(3123010434)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Transport, }
        conn id: 9, flow_id: SW:9, sibling_flags 80000000, crypto map: Tunnel1-head-0
        sa timing: remaining key lifetime (k/sec): (4187322/3312)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)





