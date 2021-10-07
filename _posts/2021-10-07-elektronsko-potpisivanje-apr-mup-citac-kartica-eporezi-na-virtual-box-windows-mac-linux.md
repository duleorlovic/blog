---
layout: post
---

Evo kako mozete da koristite elektronsko potpisivanje na bilo kom racunaru
(linux ubuntu mac windows). Od hardvera vam je dovoljno samo da imate slobodnog
hard diska i memorije (nisam siguran za minimum, ja sam imao 100GB slobodno na
eksternom hard disku i 16GB RAM memorije) i usb biometrijski citac kartica koje
se lepo prepoznaje u virtual boxu (meni radi *Yankee YCR 101*) (znam da ne radi
*Trust primo smart card reader*).

Skinuti windows koji cemo pokrenuti unutar virtual masine uz pomoc
VirtualBox:

* VirtualBox za vas sistem
[https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)
* Windows 10 VirtualBox image
  [https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/)


Otpakovati `WinDev2108Eval.VirtualBox.zip` i pomocu VirtualBox programa otvoriti
dobijeni fajl `WinDev2108Eval.ova`.

Kada se pokrene virtualna masina attachovati usb smart card reader na nju (to
treba da radite svaki put kada se pokrene masine).

![virtualbox_attach_smart_card_reader]({{ site.baseurl }}/assets/posts/virtualbox_attach_smart_card_reader.jpg "virtualbox_attach_smart_card_reader")

Mozete provetiti da li se uredjaj prepoznao na windowsu tako sto kliknete desni
klik na Windows ikonu pa na *Device Manager* i tamo bi trebalo da stoji *Smart
card readers* i *Smart cards* (ukoliko ste ubacili licnu kartu u citac, obicno
cip stoji na gore, a slika prema dole).

![virtualbox_open_device_manager.png]({{ site.baseurl }}/assets/posts/virtualbox_open_device_manager.png "virtualbox_open_device_manager.png")
![virtualbox_device_manager.png]({{ site.baseurl }}/assets/posts/virtualbox_device_manager.png "virtualbox_device_manager.png")


Sa sajta ustanove od koje ste uzeli certifikat treba da skinete midleware. Ja
sam vadio sertifikat u MUP-u tako da sam instalirao TrustEdge 64 bitni
`TrustEdgeID_x64` sa
[http://ca.mup.gov.rs/ca/ca_cyr/start/kes/](http://ca.mup.gov.rs/ca/ca_cyr/start/kes/)
klik na *+ ИНСТАЛАЦИЈА СОФТВЕРА ЗА ЛИЧНЕ КАРТЕ*. Mozete instalirati i Celik
aplikaciju za citanje licne karte. Sve ostale alate vec imate u `WinDev`
virtualnoj masini.

I to je to sto treba da instalirate. Posle pratite uputstva za elektronsko
potpisivanje.

