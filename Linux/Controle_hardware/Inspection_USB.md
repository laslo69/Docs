Répertorie les périphériques USB actuellement connectés à la machine. Même s’il existe des périphériques USB pour presque tous les usages imaginables, l’interface USB est utilisée principalement pour connecter des périphériques d’entrée - claviers, dispositifs de pointage - et des supports de stockage amovibles

La commande `lsusb` est similaire à `lspci`, mais liste exclusivement les informations USB

```bash
root@debian:~# lsusb 
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 003: ID 058f:6366 Alcor Micro Corp. Multi Flash Reader
Bus 001 Device 005: ID 1038:1216 SteelSeries ApS SteelSeries SC2 USB Headset
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 003 Device 002: ID 046a:c122 CHERRY CHERRY USB Keyboard
Bus 003 Device 003: ID 1038:183c SteelSeries ApS SteelSeries Rival 5
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
```

***

L’option `-v` affiche des informations plus détaillées.

Un périphérique spécifique peut être sélectionné pour l’inspection en fournissant son ID à l’option `-d`.

Le paramètre `-v` doit être mis avant `-d`

```bash
lsusb -v -d 1781:0c9f
Bus 001 Device 029: ID 1781:0c9f Multiple Vendors USBtiny
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               1.01
  bDeviceClass          255 Vendor Specific Class
  bDeviceSubClass         0
  bDeviceProtocol         0
  bMaxPacketSize0         8
  idVendor           0x1781 Multiple Vendors
  idProduct          0x0c9f USBtiny
  bcdDevice            1.04
  iManufacturer           0
  iProduct                2 USBtiny
  iSerial                 0
  bNumConfigurations      1
```

***

Avec l’option `-t`, la commande `lsusb` affiche le mappage en des périphériques USB sous forme d’une arborescence

```bash
lsusb -t
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=dwc_otg/1p, 480M
    |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/4p, 480M
        |__ Port 1: Dev 3, If 0, Class=Hub, Driver=hub/3p, 480M
            |__ Port 2: Dev 11, If 1, Class=Human Interface Device, Driver=usbhid, 1.5M
            |__ Port 2: Dev 11, If 0, Class=Human Interface Device, Driver=usbhid, 1.5M
            |__ Port 3: Dev 20, If 0, Class=Wireless, Driver=btusb, 12M
            |__ Port 3: Dev 20, If 1, Class=Wireless, Driver=btusb, 12M
            |__ Port 3: Dev 20, If 2, Class=Application Specific Interface, Driver=, 12M
            |__ Port 1: Dev 7, If 0, Class=Vendor Specific Class, Driver=lan78xx, 480M
        |__ Port 2: Dev 28, If 0, Class=Human Interface Device, Driver=usbhid, 1.5M
        |__ Port 3: Dev 29, If 0, Class=Vendor Specific Class, Driver=, 1.5M
```

Néanmoins, il y a des informations importantes dans les résultats fournis par `lsusb -t`

Lorsqu’un module approprié existe, son nom apparaît à la fin de la ligne correspondant au périphérique, comme dans `Driver=btusb`

La `Class` du périphérique identifie la catégorie générale, comme `Human Interface Device`, `Wireless` ou `Mass Storage`, entre autres