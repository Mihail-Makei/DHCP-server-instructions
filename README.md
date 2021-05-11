# DHCP-server-instructions
## Настройка простейшего DHCP-сервера
1. Определяем название ethernet-соединения, по которому будем настраивать DHCP-сервер. Для этого делаем
```
ip a
```
После чего получаем что-то типа
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 54:04:a6:67:be:58 brd ff:ff:ff:ff:ff:ff
    inet 93.175.20.42/24 brd 93.175.20.255 scope global dynamic enp2s0
       valid_lft 5294sec preferred_lft 5294sec
    inet6 fe80::5604:a6ff:fe67:be58/64 scope link 
       valid_lft forever preferred_lft forever
3: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 54:04:a6:67:be:59 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.0/24 scope global enp3s0
       valid_lft forever preferred_lft forever
    inet6 fe80::5604:a6ff:fe67:be59/64 scope link 
       valid_lft forever preferred_lft forever
```
В нашем случае мы используем соединение ```enp3s0```.
2. Для начала необходимо настроить статический IP-адрес для компьютера, выступающего в роли DHCP-сервера. Для этого открываем файл ```/etc/network/interfaces``` (делать это необходимо с правами суперпользователя).
```
sudo nano /etc/network/interfaces
```
После чего добавляем в него следующие строки:
```
allow-hotplug enp3s0
iface enp3s0 inet static
        address 192.168.1.0
        netmask 255.255.255.0
```
Строка ```allow-hotplug enp3s0``` в данной инструкции соответстует просто использованию данного соединения при подключении. Далее в последующих трёх строках устанавливается адрес и сетевая маска для данного соединения.
3. Далее устанавливаем пакет ```isc-dhcp-server```.
```
sudo apt-get install isc-dhcp-server
```
4. Далее редактируем файл ```/etc/dhcp/dhcpd.conf``` (предварительно делая бэкап, чтобы была возможность восстановиться).
```
sudo cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.bak
sudo nano /etc/dhcp/dhcpd.conf
```
В этом файле помещаем следующие строки
```
default-lease-time 600;
max-lease-time 7200;

subnet 192.168.1.0 netmask 255.255.255.0 {
range 192.168.1.10 192.168.1.100;
}
```
Строки ```default-lease-time``` и ```max-lease-time``` соответствуют времени, на которое выделяется IP-адрес; строка ```subnet 192.168.1.0 netmask 255.255.255.0``` устанавливает подсеть нашего IP-сервера, а заключённая в фигурные скобки строка ```range 192.168.1.10 192.168.1.100``` устанавливает диапазон IP-адресов, которые может раздавать наш DHCP-сервер.
5. Теперь редактируем файл ```/etc/default/isc-dhcp-server```, добавляя туда строку
```
INTERFACES="enp3s0"
```
6. Для запуска сервера используем команду
```
sudo systemctl start isc-dhcp-server
```
