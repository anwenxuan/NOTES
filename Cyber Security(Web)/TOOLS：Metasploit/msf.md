**msf信息收集**

Nmap扫描

​     \- db_nmap -sV192.168.1.0/24

Auxiliary 扫描模块

​     -RHOSTS<>RHOST

​      192.168.1.20-192.168.1.30、192.168.1.0/24,192168.11.0/24

​      file:/root/h.txt

​     \- search arp

​       use auxiliary/scanner/discovery/arp_sweep

​      set INTERFACE、RHOSTS、SHOST、SMAC、THREADS; run

   search portscan

​      use auxiliary/scanner/portscan/syn

​    set INTERFACE、PORTS、RHOSTS、THREADS; run