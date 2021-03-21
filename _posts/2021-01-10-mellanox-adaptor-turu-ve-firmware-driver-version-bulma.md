---
layout: post
title: "Mellanox Adaptör Türü ve Firmware/Driver version Bulma"
tags: [Mellanox, firmware, install]
---

{% highlight bash %}
lspci | grep Mellanox
13:00.0 InfiniBand: Mellanox Technologies MT27500 Family [ConnectX-3]
{% endhighlight %}

{% highlight bash %}
lspci -vv -s 13:00.0 | grep "Part number" -A 3
[PN] Part number: 7046442              
[EC] Engineering changes: XX
[SN] Serial number: ****-********      
[V0] Vendor specific: PCIe Gen3 x8
{% endhighlight %}

Mellanox firmware upgrade vs işleri için mlxup uygulamasını indiriyoruz https://www.mellanox.com/support/firmware/mlxup-mft

{% highlight bash %}
wget http://www.mellanox.com/downloads/firmware/mlxup/4.15.2/SFX/linux_x64/mlxup
chmod a+x mlxup
{% endhighlight %}

firmware versiyonumuzu kontrol ediyoruz

{% highlight bash %}
./mlxup 
Querying Mellanox devices firmware ...

Device #1:
----------

  Device Type:      ConnectX3
  Part Number:      7046442_Ax
  Description:      Oracle Dual port; ConnectX-3 ; Infiniband Adapter FDR/QDR
  PSID:             ORC1090120019
  PCI Device Name:  0000:13:00.0
  Port1 GUID:       ****************
  Port2 GUID:       ****************
  Versions:         Current        Available     
     FW             2.35.5538      N/A           

  Status:           No matching image found
{% endhighlight %}

eğer düşükse yine mlxup komutuyla güncelliyoruz

{% highlight bash %}
./mlxup -i fw-ConnectX3-rel-2_35_5538-15-7046442_7092757.bin
{% endhighlight %}

## Mellanox Firmware tools (MFT) Kurulumu

Kurulum için gerekli paketleri kuruyoruz

{% highlight bash %}
apt install graphviz make gcc quilt libltdl-dev debhelper chrpath automake autotools-dev dpatch m4 swig dkms

wget http://download.proxmox.com/debian/pve/dists/buster/pvetest/binary-amd64/pve-headers-5.4.78-2-pve_5.4.78-2_amd64.deb

dpkg -i pve-headers-5.4.78-2-pve_5.4.78-2_amd64.deb 
{% endhighlight %}

Mellanox web sitesinden MFT paketini indirelim http://www.mellanox.com/page/management_tools

{% highlight bash %}
mkdir mft

cd mft

wget https://www.mellanox.com/downloads/MFT/mft-4.15.1-9-x86_64-deb.tgz 

tar xzvf mft-4.15.1-9-x86_64-deb.tgz

./install.sh
-I- Removing mft external packages installed on the machine
-I- Installing package: /root/mft/mft-4.15.1-9-x86_64-deb/SDEBS/kernel-mft-dkms_4.15.1-9_all.deb
-I- Installing package: /root/mft/mft-4.15.1-9-x86_64-deb/DEBS/mft_4.15.1-9_amd64.deb
-I- In order to start mst, please run "mst start".
{% endhighlight %}

Başlangıç ​​olarak, MFT’yi çalıştırmamız gerekir:

{% highlight bash %
mst start
Starting MST (Mellanox Software Tools) driver set
Loading MST PCI module - Success
Loading MST PCI configuration module - Success
Create devices

mst status
MST modules:
------------
    MST PCI module loaded
    MST PCI configuration module loaded

MST devices:
------------
/dev/mst/mt4099_pciconf0         - PCI configuration cycles access.
                                   domain:bus:dev.fn=0000:13:00.0 addr.reg=88 data.reg=92 cr_bar.gw_offset=-1
                                   Chip revision is: 01
/dev/mst/mt4099_pci_cr0          - PCI direct access.
                                   domain:bus:dev.fn=0000:13:00.0 bar=0xc7600000 size=0x100000
                                   Chip revision is: 01
{% endhighlight %}                                   