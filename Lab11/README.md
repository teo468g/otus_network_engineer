## 1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path)

Настроим filter-list для R14  

router bgp 1001
neighbor 192.168.1.1 filter-list 1 out

ip as-path access-list 1 permit ^$

R14# sh ip bgp nei 192.168.1.1 advertised-routes  
BGP table version is 22, local router ID is 10.0.0.14  

     Network          Next Hop            Metric LocPrf Weight Path  
 *>  10.0.0.14/32     0.0.0.0                  0         32768 i  
 *>  172.16.4.0/24    0.0.0.0                  0         32768 ?  
 *>  172.16.5.0/24    0.0.0.0                  0         32768 ?  
 *>  172.16.6.0/24    0.0.0.0                  0         32768 ?  
 *>  172.16.7.0/24    0.0.0.0                  0         32768 ?  
 *>  192.168.1.0/31   0.0.0.0                  0         32768 i  



## 2. Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list).  

R18# router bgp 2042 
neighbor 192.168.1.7 prefix-list LIST_OUT1 out  

ip prefix-list LIST_OUT1 seq 10 permit 192.168.1.4/31
ip prefix-list LIST_OUT1 seq 15 permit 10.0.0.15/32  
ip prefix-list LIST_OUT1 seq 20 permit 172.17.0.0/16  
   
## 3. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.  

router bgp 101  
 neighbor 192.168.1.0 default-originate  
 neighbor 192.168.1.0 soft-reconfiguration inbound 
 neighbor 192.168.1.0 prefix-list DEF_OUT1 out

 ip prefix-list DEF_OUT1 seq 10 permit 0.0.0.0/0    

В Москве получаем только дефолт:  

 R14#show ip bgp nei 192.168.1.1 received-routes  
BGP table version is 30, local router ID is 10.0.0.14  

     Network          Next Hop            Metric LocPrf Weight Path  
 *   0.0.0.0          192.168.1.1                            0 101 i  

 ## 4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург.  

 Создадим дефолтный маршрут  

 R21# ip route 0.0.0.0 0.0.0.0 192.168.1.2

 Передаем его и сеть офиса в СПБ в  BGP

R21# router bgp 301  
  network 0.0.0.0  
  network 172.17.0.0    

Создадим префикс-лист для данных сетей  

R21# neighbor 192.168.1.2 prefix-list DEF_OUT1 out  
ip prefix-list DEF_OUT1 seq 10 permit 0.0.0.0/0  
ip prefix-list DEF_OUT1 seq 20 permit 172.17.0.0/16  

R15(config-router)#nei 192.168.1.3 soft-reconfiguration inbound 
R15#sh ip bgp neighbors 192.168.1.3 received-routes  

     Network          Next Hop            Metric LocPrf Weight Path  
 *>  0.0.0.0          192.168.1.3              0             0 301 i  
 *>  172.17.0.0       192.168.1.3              0             0 301 i  

 В москве получаем только дефолт и префикс офиса С.-Петербург.

