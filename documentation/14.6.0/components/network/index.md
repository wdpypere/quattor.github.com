---
layout: documentation
title: network
category: documentation
subcategory: 14.6.0/components
menu: 'components.md'
---
### NAME

network: Configure Network Settings

### DESCRIPTION

Working type definitions can be found in the README file.

The _network_ component sets the network settings through `/etc/sysconfig/network` and the various files in `/etc/sysconfig/network`-scripts. Currently only support eth\* devices.

For restarting, a sleep value of 15 is used to make sure the restarted network is fully restarted (routing may need some time to come up completely). Because of this, adding/changing lots of things may cause some slowdown.
New/changed settings are first tested by probing the CDB server on the port where the profile should be found. If this fails, the previous settings are reused.

### RESOURCES

- `/system/network/realhostname`
- `/system/network/hostname`
- `/system/network/domainname`
- `/system/network/default`\_gateway
- `/system/network/guess`\_default\_gateway
- `/system/network/gatewaydev`
- `/system/network/nisdomain`
- `/system/network/nozeroconf`

These values are used to generate the `/etc/sysconfig/network` file. ("realhostname", "guess\_default\_gateway", "default\_gateway", "gatewaydev", "nisdomain" and "nozeroconf" are optional).
By setting guess\_default\_gateway, when default\_gateway is not set, the component will try to guess the default gateway using the first configured gateway set on an interface (old style network config). (The default is true for backward compatible behaviour.)

- `/system/network/interfaces`/\[dev\]\[i\]/ip
- `/system/network/interfaces`/\[dev\]\[i\]/netmask
- `/system/network/interfaces`/\[dev\]\[i\]/broadcast
- `/system/network/interfaces`/\[dev\]\[i\]/bootproto
- `/system/network/interfaces`/\[dev\]\[i\]/onboot
- `/system/network/interfaces`/\[dev\]\[i\]/type
- `/system/network/interfaces`/\[dev\]\[i\]/device
- `/system/network/interfaces`/\[dev\]\[i\]/master

These values are used to generate the `/etc/sysconfig/network`-scripts/ifcfg-\[dev\]\[i\] files. ( "onboot", "bootproto", "master", "type" and "device" are optional; by default bootproto = static, onboot = yes, type = ethernet and device is the name of the interface used in the templates.)

- `/system/network/interfaces/eth`\[i\]/route/\[j\]/gateway
- `/system/network/interfaces/eth`\[i\]/route/\[j\]/address
- `/system/network/interfaces/eth`\[i\]/route/\[j\]/netmask

These values are used to generate the `/etc/sysconfig/network`-scripts/route-eth\[i\] files as used by ifup-routes. gateway and address should contain numerical values only. If no netmask value is given, 255.255.255.255 is used.

- `/system/network/interfaces/eth`\[i\]/aliases/\[name\]/ip
- `/system/network/interfaces/eth`\[i\]/aliases/\[name\]/netmask
- `/system/network/interfaces/eth`\[i\]/aliases/\[name\]/broadcast

These values are used to generate the `/etc/sysconfig/network`-scripts/ifcfg-eth\[i\]:\[name\] files as used by ifup-aliases.

- `/system/network/interfaces/eth`\[i\]/aliases/\[name\]/name

This value will be used to name the alias instead of \[name\]

- `/system/network/interfaces/eth`\[i\]/offload/tso

Set the TCP segment offload parameter to "off" or "on"

- `/system/network/interfaces/eth`\[i\]/ring/\[rt\]x

Set the ethernet transmit or receive buffer ring counts.  See ethtool --show-ring for the values.

- `/system/network/interfaces/eth`\[i\]/ethtool/wol

Set the wake-on-lan parameter.  See ethtool for more details of the choices.  "d" disables the
wake-on LAN.

### NOZEROCONF

Setting nozeroconf to true stops an interface from being assigned an automatic address in the 169.254. subnet.

### HWADDR

Explicitly set the MAC address for the interfaces in the configuration files. The MAC address is taken from `/hardware/cards/nic/eth`\[i\]/hwaddr.

- `/system/network/set`\_hwaddr : boolean

Set the default behaviour for all interfaces. The component default is false.

- `/system/network/interfaces/eth`\[i\]/set\_hwaddr : boolean

Set the behaviour for interface eth\[i\]. This overrides the default setting.

### CHANNEL BONDING

(see &lt;kernel&gt;/Documentation/networking/bonding.txt for more info on the driver options)

To enable channel bonding with quattor using devices eth0 and eth1 to form bond0, proceed as follows:

include pro\_software\_component\_network;


"/system/network/interfaces/eth0/bootproto"="none";
"/system/network/interfaces/eth0/master"="bond0";

"/system/network/interfaces/eth1/bootproto"="none";
"/system/network/interfaces/eth1/master"="bond0";

"/system/network/interfaces/bond0" = NETWORK\_PARAMS;
"/system/network/interfaces/bond0/driver" = "bonding";


include pro\_software\_component\_modprobe;
"/software/components/modprobe/modules" = push(nlist("name","bonding","alias","bond0"));
"/software/components/modprobe/modules" =
push(nlist("name","bonding","options","mode=6 miimon=100"));

"/software/components/network/dependencies/pre" = push("modprobe");

### VLAN support

Use the vlan\[0-9\]{0-4} interface devices and set the the explicit device name and physdev.
The VLAN ID is the number of the '.' in the device name. Physdev is mandatory for vlan\[0-9\]{0-4} device.
An example:
"/system/network/interfaces/vlan0" = VLAN\_NETWORK\_PARAMS;
"/system/network/interfaces/vlan0/device" = "eth0.3";
"/system/network/interfaces/vlan0/physdev" = "eth0";
