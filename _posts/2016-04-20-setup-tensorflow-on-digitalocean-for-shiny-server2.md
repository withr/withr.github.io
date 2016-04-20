---
layout: post
title: "Build a Compact Raspberry Pi 3 cluster (part 1) assemble"
date: 2016-04-20 16:10
comments: true
categories: RPi Big-data
---



Shopping list: 

- 4 x [Raspberry Pi 3 (model B)](https://item.taobao.com/item.htm?spm=a1z09.2.0.0.CGJIep&id=527525039334&_u=kah4vl98525)
- 4 x [SanDisk Micro SD cards (64G C10)](https://detail.tmall.com/item.htm?id=42352368230&spm=a1z09.2.0.0.CGJIep&_u=kah4vl91de6&sku_properties=5919063:6536025)
- 4 x [Micro USB power cables](https://item.taobao.com/item.htm?spm=a1z09.2.0.0.CGJIep&id=45454260322&_u=kah4vl98d59)
- 1 x [6-port USB power hub (5v12A)](https://item.taobao.com/item.htm?spm=a1z09.2.0.0.CGJIep&id=520123544343&_u=kah4vl959a9)
- 1 x [5-port network switch (hub)](https://detail.tmall.com/item.htm?id=522092553896&spm=a1z09.2.0.0.CGJIep&_u=kah4vl917f5)
- 1 x [Stacked Acrylic Case](https://item.taobao.com/item.htm?spm=a230r.1.14.39.w7a7cT&id=530438010654&ns=1&abbucket=9)
- 4 x Ethernet cables (I made them myself)


The RPi 3 consumes more power than it’s predecessor due to it includes WiFi and Bluetooth. The official suggest of power supply is no less than 800mA, but to run it safely, it’s recommended to use a 2.5A USB charger! A USB charger with one port and the maximum output current is 2.5A is common, like iPad’s charger, but a 4-ports USB charger, all ports with maximum output current 2.5A is very rare. I spent a lot time, and finally found this cheap yet meet the requirement USB power hub. It has 6 USB ports, and maximum output current can reach to 12A (the seller said 10A is stable,while 12A can’t last long ). 

![]( /images/Cluster/usbPowerHub.jpg )

The two extra USB ports can be used to supply a fan and the network switch (hub), they consume around 0.7A together. 

You can buy the stacked Acrylic case from a web shop, however, I designed mine because I feel the space between two layers should be bigger. Here is the Acrylic layer (3mm) I designed, you can download the [dwg](/images/Cluster/RPiShelf.dwg) document, and ask [somebody](https://store.taobao.com/shop/view_shop.htm?spm=a1z09.2.0.0.CGJIep&user_number_id=42034200) who has a laser cutter to make them (5x) for you.  

![]( /images/Cluster/RPiShelf.png )


My stacked shelf also need the following bolts and nuts:

- 25 x [Nylon bolt (M2.5x6)](https://detail.tmall.com/item.htm?id=523889606915&spm=a1z09.2.0.0.KobEgd&_u=kah4vl98b5d)
- 25 x [Nylon pillar (M2.5x5+6)](https://detail.tmall.com/item.htm?id=23328004099&spm=a1z09.2.0.0.KobEgd&_u=kah4vl91bef)
- 25 x [Nylon nut (M2.5)](https://detail.tmall.com/item.htm?spm=a1z10.5-b.w4011-2672328351.146.ZFnr4Z&id=43513624091&rn=9bd37717e40e72e78b1b04b51ee957a2&abbucket=13)
- 4 x [Copper pillar (M3X5+6)](https://detail.tmall.com/item.htm?id=15597886091&spm=a1z09.2.0.0.KobEgd&_u=kah4vl905e7)
- 16 x [Copper pillar (M3X25+6)](https://detail.tmall.com/item.htm?id=15597886091&spm=a1z09.2.0.0.KobEgd&_u=kah4vl905e7)
- 4 x [Copper nut (M3)](https://detail.tmall.com/item.htm?id=22079879856&spm=a1z09.2.0.0.KobEgd&_u=kah4vl92b20)


The assemble is quite easy, I think you can understand by looking at the following pictures: 

![]( /images/Cluster/20160418_220058-1.jpg )

![]( /images/Cluster/20160418_221029-1.jpg )

![]( /images/Cluster/20160418_222309-1.jpg )

![]( /images/Cluster/20160419_232027-1.jpg )

![]( /images/Cluster/20160419_232051-1.jpg )

![]( /images/Cluster/20160419_232040-1.jpg )


