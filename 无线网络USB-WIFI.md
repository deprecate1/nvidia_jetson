目前淘宝卖的最多的几款USB无线网卡：

|               | 价格 |                                                              |
| ------------- | ---- | ------------------------------------------------------------ |
| 水星Mw150us   | 29   | 芯片是RT8188GU，linux驱动很难找调了2天没搞好，放弃           |
| 腾达U9        | 49   | nano完美驱动（内置驱动及开源驱动都很稳定）<br>11n：200Mbps；11ac：433Mbps；支持2.4G和5G；<br>支持station和ap模式 |
| dlink DWA 171 | 120  | 600M双频，价格太贵，放弃                                     |

强烈建议使用腾达U9 ，价格便宜，性能稳定，功能强大，出货量巨大，不存在买不到的情况



(1)接上腾达U9  usb wifi

lsusb得到：

	nv@nv-desktop:~$ lsusb 
	Bus 001 Device 003: ID 0bda:1a2b Realtek Semiconductor Corp.

VID/PID是0bda:1a2b是一个cdrom，现在的无线网卡流行storage mass模式，执行下面指令切换成网卡模式：


	sudo  usb_modeswitch  -KW  -v  0bda  -p  1a2b
	会将usb的状态从storage mass模式切换成网卡模式，然后再lsusb得到正确vid/pid
	
	nv@nv-desktop:~$ lsusb 
	Bus 001 Device 004: ID 0bda:c811 Realtek Semiconductor Corp.

VID/PID是0bda:c811，现在可以来驱动它了，rtl8211cu兼容rt8111cu，建议用系统自带驱动，也可以gihub搜rtl8821CU

	modprobe 8821cu

现在ifconfig可以看到存在wlan0设备



(2)系统其它设置

	sudo  systemctl  disable  wpa_supplicant
	sudo  systemctl  mask     wpa_supplicant

把自带的wpa服务关闭，用我们自己的wpa进程

	iw  list
	iw  wlan0  scan
	iw  wlan0  info                   可以查看是sta模式还是ap模式



(3)sta模式

	wpa_supplicant -i wlan0 -c /etc/wpa_supplicant.conf
	上面的wpa_supplicant.conf不需要更改，只是把wpa客户端服务启动而已，然后执行：
	iwconfig wlan0 mode managed
	wpa_cli -i wlan0 set_network 0 ssid '"fee"'
	wpa_cli -i wlan0 set_network 0 psk '"Zbyy12345678@"'
	wpa_cli -i wlan0 enable_network 0
	udhcpc -i wlan0 -s /etc/udhcpc/default.script -q
	wpa_cli -i wlan0 save_config
注意上面的ssid和psk必须用单引加双引，否则失败



(4)ap模式

	sudo apt-get install hostapd  dnsmasq

配置hostapd

	cp   /etc/default/hostapd  /etc/hostapd/hostapd.conf

配置/etc/hostapd/hostapd.conf在最后面追加：

	interface=wlan0
	ssid=nano_04211190237170400202
	channel=6
	hw_mode=g
	ignore_broadcast_ssid=0
	auth_algs=1
	wpa=3
	wpa_passphrase=12345678
	wpa_key_mgmt=WPA-PSK
	wpa_pairwise=TKIP
	rsn_pairwise=CCMP

配置/etc/dnsmasq.conf在最后面追加：

	interface=wlan0
	bind-interfaces
	listen-address=10.0.0.1
	dhcp-range=10.0.0.110,10.0.0.190,255.255.255.0,12h

执行指令：

	hostapd /etc/hostapd/hostapd.conf
	dnsmasq

现在ping百度，能够回应ip地址说明dns正常；但不能回应数据说明路由不正常；配置iptables：

	iptables -F
	iptables -X
	iptables -t nat -F
	iptables -t nat -X
	iptables -t nat -APOSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE
	iptables -A FORWARD -s 10.0.0.0/24 -o eth0 -j ACCEPT
	iptables -A FORWARD -d 10.0.0.0/24 -m conntrack --ctstate ESTABLISHED,RELATED -i eth0 -j ACCEPT

验证：

	netstat -anulp | grep :53
	netstat -anulp | grep :67

ap模式就是要把这三个地方搞通：dhcp服务；dns服务；路由



关于ap模式下的ssid，可以使用这个值：cat /sys/firmware/devicetree/base/serial-number





