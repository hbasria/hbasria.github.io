---
layout: post
title: "Linux Sunucularda LVM Disk Oluşturma/Genişletme"
tags: [Linux, LVM, Disk]
---

Bir Linux sistemi kurarken en önemli kararlardan biri, sistem dosyaları, ana dizinler ve diğerleri için ayrılacak depolama alanı miktarıdır. Bu noktada bir hata yaparsanız, alan kalmamış bir bölümü büyütmek külfetli ve biraz riskli olabilir.

## Fiziksel Birimler (Physical Volumes), Hacim Grupları(Volume Groups) ve Mantıksal Birimler(Logical Volumes) Oluşturma

/dev/sdb, /dev/sdc ve /dev/sdd üzerinde fiziksel birimler oluşturmak için şunları yapın:

{% highlight bash %}
pvcreate /dev/sdb /dev/sdc /dev/sdd
{% endhighlight %}

Yeni oluşturulan PV'leri şu şekilde listeleyebilirsiniz:

{% highlight bash %}
pvs
{% endhighlight %}

/dev/sdb, /dev/sdc kullanarak vg0 adlı bir birim grubu oluşturmak için (başka aygıtlar ekleme durumunu göstermek için /dev/sdd'yi daha sonra kaydedeceğiz):

{% highlight bash %}
vgcreate vg0 /dev/sdb /dev/sdc
{% endhighlight %}

Volume Groupları görmek için:

{% highlight bash %}
vgdisplay vg0
{% endhighlight %}

Örneğin, sırasıyla proje dosyalarını ve sistem yedeklerini depolamak için kullanabileceğimiz vol_projects ve vol_backups adlı iki LV oluşturalım.

{% highlight bash %}
lvcreate -n vol_projects -L 10G vg0
lvcreate -n vol_backups -l 100%FREE vg0
{% endhighlight %}

Her mantıksal birim kullanılmadan önce, üstüne bir dosya sistemi oluşturmamız gerekir.

{% highlight bash %}
mkfs.ext4 /dev/vg0/vol_projects
mkfs.ext4 /dev/vg0/vol_backups
{% endhighlight %}

## Mantıksal Birimleri Yeniden Boyutlandırma ve Hacim Gruplarını Genişletme

Şimdi vol_projects'te bol miktarda alan varken vol_backups'da alanınız tükenmeye başlıyor. LVM'nin doğası gereği, her dosya sistemini aynı anda yeniden boyutlandırırken ikincisinin boyutunu kolayca küçültebilir ve birincisi için tahsis edebiliriz.

{% highlight bash %}
lvreduce -L -2.5G -r /dev/vg0/vol_projects
lvextend -l +100%FREE -r /dev/vg0/vol_backups
{% endhighlight %}

Mantıksal bir birimi yeniden boyutlandırırken eksi (-) veya artı (+) işaretlerini dahil etmek önemlidir. Aksi takdirde, LV'yi yeniden boyutlandırmak yerine sabit bir boyut ayarlarsınız.

İlk kurulumumuzdan (/dev/sdd) kalan PV'yi ekleyelim.

/dev/sdd'yi vg0'e eklemek için:

{% highlight bash %}
vgextend vg0 /dev/sdd
{% endhighlight %}

Artık yeni eklenen alanı ihtiyaçlarınıza göre mevcut LV'leri yeniden boyutlandırmak veya gerektiğinde yenilerini oluşturmak için kullanabilirsiniz.

## Mantıksal Birimleri Önyükleme aşamasında yükleme

{% highlight bash %}
blkid /dev/vg0/vol_projects
blkid /dev/vg0/vol_backups
{% endhighlight %}

Her LV için bağlama noktaları oluşturun:

{% highlight bash %}
mkdir /home/projects
mkdir /home/backups
{% endhighlight %}

Ve /etc/fstab içine karşılık gelen girişleri ekleyin (blkid ile elde edilen UUID'leri kullandığınızdan emin olun):

{% highlight bash %}
UUID=b85df913-580f-461c-844f-546d8cde4646 /home/projects	ext4 defaults 0 0
UUID=e1929239-5087-44b1-9396-53e09db6eb9e /home/backups ext4	defaults 0 0
{% endhighlight %}

Ardından değişiklikleri kaydedin ve LV'leri bağlayın:

{% highlight bash %}
mount -a
mount | grep home
{% endhighlight %}

# Kaynak

https://www.tecmint.com/manage-and-create-lvm-parition-using-vgcreate-lvcreate-and-lvextend/
