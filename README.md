# pm
Ecorouter:
hostname ISP
ip domain-name Фамилия Имя
Ip name-server 77.88.8.8
Ip name-server 192.168.1.1 ( можно любой другой айпишник )
Ip route 0.0.0.0/0 10.51.51.1 ( смотри в основном шлюзе Ethernet Ethernet)

int nat
Ip nat outside
Ip addr 10.51.51.100/24 ( придумай )
Ex
Int lc
Ip nat inside
Ip addr 192.168.2.1/24 ( опять же, придумай )
Ex
port ge0
Service-instance nat
Encapsulation untagged
Connect ip int nat
Ex
port ge1
Service-instance lc
Encapsulation untagged
Connect ip int lc
Ex
Ip nat pool nat1 192.168.2.2-192.168.2.50
Security none
ip nat source dynamic inside-to-outside pool nat1 overload interface nat
admc srv:
ip addr: 192.168.2.3
шлюз:192.168.2.1
DNS: 8.8.8.8
Проверяем пинг 8.8.8.8
Apt-get update
Apt-get install task-samba-dc
Rm -rf /etc/samba/smb.conf
Rm -rf /var/cache/samba
Rm -rf /var/lib/samba/sysvol
Samba-tool domain provision
Realm: Ryabov.Yury
Enter x3
P@ssw0rd
Cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
Cd /etc/net/ifaces/ens33
Vim resolv.conf
Nameserver 127.0.0.1
Search Ryabov.Yury
Wq
Systemctl enable –now samba.service
Samba-tool dns zonelist 127.0.0.1 -U Administrator
P@ssw0rd
Dhcp:
Cd /etc/net/ifaces/ens33
Vim resolv.conf
192.168.2.3
Search Ryabov.Yury
Wq
Systemctl restart network
Apt-get update
Ping admc
Ssh admin@192.168.2.1
Yes
Admin ( это пароль от эко роутера )
Apt-get install dhcp-server
Systemctl enable –now dhcpd
Cd /etc/dhcp 
Cp dhcpd.conf.sample dhcpd.conf
Vim dhcpd.conf
subnet 192.168.2.0
option routers 192.168.2.1
убираем полностью option nis-domain
в option domain-name меняем на Ryabov.Yury
domain-name-servers 192.168.2.3
range dynamic-bootp 192.168.2.4 192.168.2.10
под max-lease-time жмем enter x2 
host CLI {
hardware ethernet и мак адрес клиента ( запускаем его, жмем Network Adapter и Advanced);
fixed-address 192.168.2.5;
}
Wq
Dhcpd -t
Systemcrl restart dhcpd
Systemctl status dhcpd
CLI:
Открываем консоль, заходим в рута
Apt-get update
Apt-get install -y task-auth-ad-sssd gpui gpupdate admc
Если ошибка один раз то ребут, если 2 раза то сносим клиент и делаем по новой
ребут
Затем заходим в “ центр управления системой, пользователи, аутентификация
Включаем Active Directory ( рабочая группа: Фамилия )
Опять ребут
Заходим в ADMC ( Administrator P@ssw0rd)
ПКМ по Ryabov.yury создать подразделение ( Ryabov)
В подразделении создать группу  и пользователя
Переносим пользователя в группу
Жмем “Объекты групповой политики” и включаем принудительно, затем пкм и изменить
В поиске пишем курсор, и разворачиваем все в пользователе
Открыть консоль
Gpupdate
Gpupdate –force
Заходим в пользователя (rya)
