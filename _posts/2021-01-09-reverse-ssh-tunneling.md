---
layout: post
title: "Reverse SSH Tunneling"
description: "NAT'ın arkasında duran Linux sunucuya ssh yapmak."
thumb_image: "documentation/sample-image.jpg"
tags: [Reverse, SSH, Tunneling]
---

## Reverse SSH Tunel Kurulumu

Erişmek istediğiniz sunucunun IP sinin 192.168.20.25 olduğunu varsayalım.

internete açık bir sunucudan bu işlemleri yaptığınızı varsayıyoruz sunucunuzun ip si 1.1.1.1 olsun

1. natın arkasında olan erişmek istediğiniz makine üzerinde aşağıdaki komutu çalıştırın

{% highlight bash %}
ssh -R 19999:localhost:22 sourceuser@1.1.1.1
{% endhighlight %}

2. internet tarafında olan sunucuda aşağıdaki komutu çalıştırarak nat arkasındaki sunucuya tunel üzerinden bağlanabilirsiniz

{% highlight bash %}
ssh localhost -p 19999
{% endhighlight %}
