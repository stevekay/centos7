# centos7

## Networking on VMware Player

No network at first boot:

    # ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether 00:0c:29:a7:8d:47 brd ff:ff:ff:ff:ff:ff
    #

To have NIC enabled via DHCP on each boot:

    # sed -i 's/ONBOOT=no/ONBOOT=yes/' /etc/sysconfig/network-scripts/ifcfg-ens*
    
Takes effect on next reboot.  For immediate effect

    # systemctl restart network

    # systemctl restart network
    # ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether 00:0c:29:a7:8d:47 brd ff:ff:ff:ff:ff:ff
        inet 192.168.74.133/24 brd 192.168.74.255 scope global dynamic ens33
           valid_lft 1799sec preferred_lft 1799sec
        inet6 fe80::d7a9:aec5:3df7:3b72/64 scope link
           valid_lft forever preferred_lft forever
    #

## Update packages

    yum update

