# Доступ к сетевым устройствам по протоколу SSH  
## Топология  
<img width="665" height="139" alt="image" src="https://github.com/user-attachments/assets/01662d89-fb76-4ea2-84cd-aadbf882c1a1" />

## Таблица адресации  


| Устройство | Интерфейс | IP-адрес      | Маска подсети    | Шлюз по умолчанию |
|------------|-----------|---------------|------------------|--------------------|
| R1         | G0/0/1     | 192.168.1.1   | 255.255.255.0    | —                  |
| S1         | VLAN 1     | 192.168.1.11  | 255.255.255.0    | 192.168.1.1        |
| PC-A       | NIC        | 192.168.1.3   | 255.255.255.0    | 192.168.1.1        |  
## Часть 1. Настройка основных параметров устройств  
#### Настраиваю устройства, чтобы построить сеть согласно топологии.
### PC-A  

#### Применяю настройки сети, проверяю через командную строку:  

C:\>ipconfig  
  
FastEthernet0 Connection:(default port)  
  
   Connection-specific DNS Suffix..:   
   Link-local IPv6 Address.........: FE80::2E0:8FFF:FEB1:7DE9  
   IPv6 Address....................: ::  
   IPv4 Address....................: 192.168.1.3  
   Subnet Mask.....................: 255.255.255.0  
   Default Gateway.................: ::  
                                     192.168.1.1  

### S-1  

#### Подключаюсь к коммутатору через консольный порт. Провожу настройку через терминал:  

Switch> enable  
Switch# configure terminal  
Enter configuration commands, one per line.  End with CNTL/Z.  
Switch(config)# no ip domain-lookup  
Switch(config)# hostname S1  
S1(config)# enable secret class  
S1(config)# line console 0  
S1(config-line)# password cisco  
S1(config-line)# login  
S1(config-line)# logging synchronous  
S1(config-line)# exec-timeout 5 0  
S1(config-line)# exit  
S1(config)# line vty 0 15  
S1(config-line)# password cisco  
S1(config-line)# login  
S1(config-line)# exec-timeout 5 0  
S1(config-line)# exit  
S1(config)# interface vlan1  
S1(config-if)# ip address 192.168.1.11 255.255.255.0  
S1(config-if)# no shutdown  
S1(config-if)#  
%LINK-5-CHANGED: Interface Vlan1, changed state to up  
%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up  
S1(config-if)# exit  
S1(config)# service password-encryption  
S1(config)# banner motd ^CGo OUT NOW!^C  
S1(config)# end  
%SYS-5-CONFIG_I: Configured from console by console  
S1# copy running-config startup-config  
Destination filename [startup-config]?  
Building configuration...  
[OK]  

### R1

#### Подключаюсь через консольный порт к маршрутизатору:  

Router> enable  
Router# configure terminal  
Router(config)# no ip domain-lookup  
Router(config)# hostname R1  
R1(config)# enable secret class  
R1(config)# line console 0  
R1(config-line)# password cisco  
R1(config-line)# login  
R1(config-line)# logging synchronous  
R1(config-line)# exec-timeout 5 0  
R1(config-line)# exit  
R1(config)# line vty 0 4  
R1(config-line)# password cisco  
R1(config-line)# login  
R1(config-line)# exec-timeout 5 0  
R1(config-line)# exit  
R1(config)# interface g0/0/1  
R1(config-if)# ip address 192.168.1.1 255.255.255.0  
R1(config-if)# no shutdown  
R1(config-if)#  
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up  
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up  
R1(config-if)# exit  
R1(config)# service password-encryption  
R1(config)# banner motd ^CGo OUT NOW!^C  
R1(config)# end  
R1# copy running-config startup-config  
Destination filename [startup-config]?  
Building configuration...  
[OK]  
R1#
  
#### Проверяю примененные настройки с ПК с помощью команды ping:  
  
C:\>ping 192.168.1.11  
Pinging 192.168.1.11 with 32 bytes of data:  
Reply from 192.168.1.11: bytes=32 time<1ms TTL=255  
Reply from 192.168.1.11: bytes=32 time<1ms TTL=255  
Reply from 192.168.1.11: bytes=32 time<1ms TTL=255  
Reply from 192.168.1.11: bytes=32 time<1ms TTL=255  
Ping statistics for 192.168.1.11:  
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),  
Approximate round trip times in milli-seconds:  
    Minimum = 0ms, Maximum = 0ms, Average = 0ms  
    
## Часть 2. Настройка маршрутизатора для доступа по протоколу SSH

### R1
Go OUT NOW!  
User Access Verification  
Password:   
R1>enable  
Password:   
R1#conf term  
Enter configuration commands, one per line.  End with CNTL/Z.  
R1(config)#ip domain-name lab05.local  
R1(config)#crypto key generate rsa  
The name for the keys will be: R1.lab05.local  
Choose the size of the key modulus in the range of 360 to 4096 for your  
  General Purpose Keys. Choosing a key modulus greater than 512 may take  
  a few minutes.  
How many bits in the modulus [512]: 2048  
% Generating 2048 bit RSA keys, keys will be non-exportable...[OK]  
R1(config)#username admin secret Adm1nP@55 privilege 15  
*Mar 1 0:30:18.948: %SSH-5-ENABLED: SSH 1.99 has been enabled  
R1(config)#line vty 0 4  
R1(config-line)#transport input ssh   
R1(config-line)#login local  
R1(config-line)#exit  
S1(config)# ip ssh version 2  
R1(config)#end  
R1#  
%SYS-5-CONFIG_I: Configured from console by console  
R1#copy running-config startup-config   
Destination filename [startup-config]?   
Building configuration...  
[OK]  
R1#  
#### С помощью SSH  устанавливаю соединение с маршрутизатором:
<img width="980" height="125" alt="{2F7F6B84-E053-4539-8555-651E83BC991F}" src="https://github.com/user-attachments/assets/8222202a-3ba4-4a00-8a5f-bb51ec209420" />


## Часть 3. Настройка коммутатора для доступа по протоколу SSH

Go OUT NOW!^  
User Access Verification  
Password: 
S1>enable  
Password: 
S1#conf term  
Enter configuration commands, one per line.  End with CNTL/Z.  
S1(config)#ip domain-name lab05.local  
S1(config)#crypto key generate rsa  
The name for the keys will be: S1.lab05.local  
Choose the size of the key modulus in the range of 360 to 4096 for your  
  General Purpose Keys. Choosing a key modulus greater than 512 may take  
  a few minutes.    
How many bits in the modulus [512]: 2048  
% Generating 2048 bit RSA keys, keys will be non-exportable...[OK]  
S1(config)#username admin secret Adm1nP@55 privilege 15  
*Mar 1 1:39:20.201: %SSH-5-ENABLED: SSH 1.99 has been enabled  
S1(config)#line vty 0 15  
S1(config-line)#transport input ssh  
S1(config-line)#login local  
S1(config-line)#exit  
S1(config)#ip ssh version 2  
S1(config)#end  
S1#  
%SYS-5-CONFIG_I: Configured from console by console   
S1#copy running-config startup-config   
Destination filename [startup-config]?   
Building configuration...  
[OK]  
#### Устанавливаю соединение с коммутатором по протоколу SSH
<img width="983" height="155" alt="{67ADB910-C9E5-42D4-8679-BE207EF44554}" src="https://github.com/user-attachments/assets/db0931df-d5ac-4109-8ddb-db9437ca2e1c" />

## Часть 4. Настройка протокола SSH с использованием интерфейса командной строки (CLI) коммутатора
#### Установка с S1 соединения с R1 по SSH  
<img width="982" height="313" alt="{FA916B0D-5FDC-40DE-8BC1-FF1EECCAD992}" src="https://github.com/user-attachments/assets/dea091c0-24fe-4d22-b117-ce1198575aac" />  
<img width="539" height="74" alt="{7FF54CCB-37E8-4BB5-861F-C3B89B313EC9}" src="https://github.com/user-attachments/assets/90d4ff73-f082-4b25-91f6-536859a76423" />  
  
## 	Вопрос для повторения  
Для предоставления доступа нескольким пользователям с уникальными именами можно использовать внешний сервер RADIUS, тогда доступ будет предоставляться до данным доменной УЗ.  
Так же можно добавить пользователя через терминал Cisco:   

C:\>ssh -l admin 192.168.1.1  
Password:  
Go OUT NOW!  
R1>enable 
Password:   
R1#conf term  
Enter configuration commands, one per line.  End with CNTL/Z.  
R1(config)#line vty 0 15  
R1(config-line)#username kola secret Admeeen1 privilege 1  
R1(config)#  

#### От уровня привилегий зависят права пользователя: 15- полный доступ, 1- только просмотр.
