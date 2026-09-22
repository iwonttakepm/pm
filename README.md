при установке isp мы указываем ip на провод
который идет к экороутеру (192.168.1.1),
шлюз мы не указываем.
НЕ ЗАБЫВАЕМ ПОМЕНЯТЬ ИМЕНА МАШИН

НА ISP:
nano /etc/resolv.conf (устанавливаем днс example [nameserver 8.8.8.8])
nano /etc/net/sysctl.conf (ставим 1 в ip_forward)
systemctl restart network
iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
iptables-save > /etc/sysconfig/iptables
iptables-restore /etc/sysconfig/iptables
systemctl enable iptables --now
nano /etc/net/ifaces/ens34/ipv4route
(из сети 10.0.0.120/30) via (должны идти на экороутер 192.168.1.2)
192.168.0.0/24 via 192.168.1.2 (маршрут для локи)
192.168.73.64/28 via 192.168.1.2

ПЕРЕХОДИМ НА ECOROUTER:
en
conf t
int toisp
ip address 192.168.1.2/24
exit
ip route 0.0.0.0/0 192.168.1.1 (шлюз) (указываем isp как шлюз)
ip route 192.168.0.0/24 10.0.0.122 (из сети локи отправляем пакет на тора)
ip route 192.168.73.64/28 10.0.0.122
exit
wr
conf t
port ge0
service-instance (название интерфейса) toisp
encapsulation untagged
connect ip interface toisp
int tothor
ip address 10.0.0.121/30
exit
port ge1
service-instance tothor
encapsulation untagged
connect ip interface tothor
ex
ex
ex
wr
УСТАНАВЛИВАЕМ THOR:
ens33 (к ecorouter - ip 10.0.0.122/30) (шлюз 10.0.0.121)
ens34 (к loki - ip 192.168.0.1/24)
ens35 (к magni,modi - ip 192.168.73.65/28)
ЗАХОДИМ НА THOR:
nano /etc/net/sysctl.conf (ставим 1 в ip_forward)
nano /etc/resolv.conf (устанавливаем днс example [nameserver 8.8.8.8])
systemctl restart network
ЗАХОДИМ НА ISP:
nano /etc/net/ifaces/ens34/ipv4route
(из сети 10.0.0.120/30) via (должны идти на экороутер 192.168.1.2)
192.168.0.0/24 via 192.168.1.2 (маршрут для локи)
192.168.73.64/28 via 192.168.1.2
ЗАХОДИМ НА THOR:
apt-get update
apt-get install dhcp-server
cd /etc/dhcp/
ls
cp dhcpd.conf.example dhcpd.conf
nano dhcpd.conf (стираем лишнее)
subnet (подсеть) 192.168.0.0 netmask 255.255.255.0 {
range 192.168.0.1 192.168.1.50;
option domain-name-servers 77.88.8.8; // после установки samba доена ip конторолера ex: 192.168.73.65
option routers (шлюз в сторону локи) 192.168.0.1;
default-lease-time 900;
max-lease-time 7200;
}
host loki {
hardware ethernet (вставляем мак адрес провода на локи);
fixed-address 192.168.0.20; (по заданию место +12)
}

systemctl restart dhcpd
systemctl status dhcpd
systemctl enable dhcpd --now
УСТАНАВЛИВАЕМ ЛОКИ:
адрес должен быть выдан dhcp
проверяем пинг
УСТАНАВЛИВАЕМ Magni:
ip 192.168.73.66
шлюз 192.168.73.65
ЗАХОДИМ НА MAGNI:
nano /etc/resolv.conf
nameserver 77.88.8.8
SAMBA
apt-get install samba-dc task-auth-ad task-samba-dc
rm -rf /etc/samba/smb.conf
nano /etc/krb5.conf заменить ВСЕ example.com на свой домен
samba-tool domain provision
lastname.name
2)enter
3)enter
4)enter
systemctl enable --now samba
systemctl status samba


УСТАНАВЛИВАЕМ Modi:
ip 192.168.73.67
шлюз 192.168.73.65
НАСТРОЙКА SSH:
заходим на тора/magni/modi
nano /etc/openssh/sshd_config
меняем:
port 22 на port 1224 (по заданию)
passwordauthentication yes
permitrootlogin yes
systemctl restart sshd
ПОТОМ ЗАХОДИМ НА ЛОКИ:
nano ~/.ssh/config
Host thor
HostName 192.168.0.1
User root
Port 1224
Host magni
HostName 192.168.73.66
User root
Port 1224
Host modi
HostName 192.168.73.67
User root
Port 1224


          ANSIBLE
/etc/ansible/hosts
[clients]
name ansible_host=ipхоста ansible_user=логин ansible_password=пароль ansible_port=порт
пример:
thor ansible_host=192.168.2.2 ansible_user=root ansible_password=1 ansible_port=1224

/etc/ansible/ansible.cfg
[defaults]
Inventory =/etc/ansible/hosts
Host_key_checking = False
проверка
ansible -m ping all
---
дебаг
если не стартует сервис (dhcpd/samba)
journalctl -xeu сервис
journalctl -xeu dhcpd
