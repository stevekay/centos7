# Adding USB wifi adapter

## syslog entries

    $ tail -6 /var/log/messages
    Feb 25 23:03:22 localhost kernel: usb 1-1: new high-speed USB device number 2 using ehci-pci
    Feb 25 23:03:23 localhost kernel: usb 1-1: New USB device found, idVendor=0bda, idProduct=8812
    Feb 25 23:03:23 localhost kernel: usb 1-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    Feb 25 23:03:23 localhost kernel: usb 1-1: Product: 802.11n NIC
    Feb 25 23:03:23 localhost kernel: usb 1-1: Manufacturer: Realtek
    Feb 25 23:03:23 localhost kernel: usb 1-1: SerialNumber: 123456
    $

## lsusb details

    $ sudo yum install -y usbutils
    [snip]
    $ sudo lsusb -s 1:2
    Bus 001 Device 002: ID 0bda:8812 Realtek Semiconductor Corp. RTL8812AU 802.11a/b/g/n/ac WLAN Adapter
    $ sudo lsusb -v -s 1:2
    
    Bus 001 Device 002: ID 0bda:8812 Realtek Semiconductor Corp. RTL8812AU 802.11a/b/g/n/ac WLAN Adapter
    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               2.00
      bDeviceClass            0 (Defined at Interface level)
      bDeviceSubClass         0
      bDeviceProtocol         0
      bMaxPacketSize0        64
      idVendor           0x0bda Realtek Semiconductor Corp.
      idProduct          0x8812 RTL8812AU 802.11a/b/g/n/ac WLAN Adapter
      bcdDevice            0.00
      iManufacturer           1 Realtek
      iProduct                2 802.11n NIC
      iSerial                 3 123456
      bNumConfigurations      1
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength           53
        bNumInterfaces          1
        bConfigurationValue     1
        iConfiguration          0
        bmAttributes         0x80
          (Bus Powered)
        MaxPower              500mA
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        0
          bAlternateSetting       0
          bNumEndpoints           5
          bInterfaceClass       255 Vendor Specific Class
          bInterfaceSubClass    255 Vendor Specific Subclass
          bInterfaceProtocol    255 Vendor Specific Protocol
          iInterface              0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x81  EP 1 IN
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x02  EP 2 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x03  EP 3 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x04  EP 4 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x85  EP 5 IN
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0040  1x 64 bytes
            bInterval               1
    Device Qualifier (for other device speed):
      bLength                10
      bDescriptorType         6
      bcdUSB               2.00
      bDeviceClass            0 (Defined at Interface level)
      bDeviceSubClass         0
      bDeviceProtocol         0
      bMaxPacketSize0        64
      bNumConfigurations      1
    Device Status:     0x0002
      (Bus Powered)
      Remote Wakeup Enabled
    $ 

## Download + compile driver

Using details from https://github.com/gnab/rtl8812au

    $ yum install -y git gcc kernel-devel
    [snip]
    $ git clone git@github.com:gnab/rtl8812au.git
    Cloning into 'rtl8812au'...
    [snip]
    $ cd rtl8812au
    $ make
    [snip]
      LD [M]  /home/steve/wifi/rtl8812au/8812au.o
      Building modules, stage 2.
      MODPOST 1 modules
      CC      /home/steve/wifi/rtl8812au/8812au.mod.o
      LD [M]  /home/steve/wifi/rtl8812au/8812au.ko
    make[1]: Leaving directory `/usr/src/kernels/3.10.0-514.6.2.el7.x86_64'
    $

## Insert module, spot new ens35u1 nic

    $ sudo insmod 8812au.ko
    $ ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether 00:0c:29:a7:8d:47 brd ff:ff:ff:ff:ff:ff
        inet 192.168.74.133/24 brd 192.168.74.255 scope global dynamic ens33
           valid_lft 1581sec preferred_lft 1581sec
        inet6 fe80::d7a9:aec5:3df7:3b72/64 scope link
           valid_lft forever preferred_lft forever
    3: ens35u1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN qlen 1000
        link/ether 24:05:0f:9e:7f:aa brd ff:ff:ff:ff:ff:ff
    $

## Copy module to /lib, regenerate modules.dep

    $ sudo cp 8812au.ko /lib/modules/$(uname -r)/kernel/drivers/net/wireless
    $ sudo depmod
    $

## Reboot, ensure nic remains

    $ shutdown -r now
    $ [root@localhost ~]# ip a s dev ens35u1
    3: ens35u1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN qlen 1000
        link/ether 24:05:0f:9e:7f:aa brd ff:ff:ff:ff:ff:ff
        inet6 fe80::6aaf:b53a:4dca:6da0/64 scope link tentative
           valid_lft forever preferred_lft forever
    $ 
