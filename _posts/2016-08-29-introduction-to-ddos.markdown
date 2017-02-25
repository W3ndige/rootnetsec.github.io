---
layout:     post
title:      "Distributed Denial of Service Attack"
subtitle:   "Introduction to DDOS Attacks"
date:       2016-08-29 6:00:00
author:     "W3ndige"
header-img: "img/introduction-to-ddos-header.png"
category: Networking
---
<p>Denial of Service attacks (DOS) are one of the simplest ways to paralylze network infrastructure of the victim. Although  many attacks are connected with destroying data or stealing credentials (or any kind of sensitive information), DOS attacks can cause equally big financial loss. Properly prepared, in particular Distributed Denial of Service, can cause catastrophic harm to the victim company. In addition - even though they are one of the earliest kinds of network attacks - they are getting more and more sophisticated and harder to prevent. </p>

<h1>(D)DOS - Evolution of Attacks</h1>

<p>DOS attacks are threats that from many years make life harder for administrators. In the 90s they were more sophisticated because of the small amount of devices connected to the Web. Back then most of the attacks used vulnerable networking software, where attackers sent properly prepared packets that suspended operating systems. They were mostly only peformed from the one node which means that they were very different from the modern ones. </p>

<p>But at the beginning of the XXI century, Internet access became more broader - giving more people ability to connect their devices to the Web. But what came with that is the rapid growth of malware hiding in the mail attachments, in software or on the infected websites. In the same time network software became more secure - to prevent attacks from growing number of people interested in hacking. That's where attackers needed more that one device to perform something called Distributed Denial of Service attack (DDOS) which used more than one node - preferably computers infected with malware.  Goal of this type of attack is flooding the victim with packets originating from hundreds of sources. </p>

<p>Nowadays, about 1/3 people on the world are connected to the Internet, where not only computers are connected but smartphones, tablets - even Smart TVs, fridges or light bulbs. That gives great opportunity for attackers who can try and exploit each of this devices - which can contain some flaws useful for an attacker. Such a big number of devices in the Internet causes the fact that most of the modern attacks are distributed. Attackers try to find vulnerabilities in devices, exploit them and add it into their botnet network. Then the bandwith of these devices is used to coordinate DDOS attack, flooding the victim with millions of packets. Botnets in these days can contain hundreds or even thousands of devices - each ready to attack causing the victims network to be flooded with packets running at speed of Gigabits per second - challenge for administrators and even ISP's - which may be affected with such a DDOS. Quite a pain in the neck, isn't it?   </p>

<h1>Kinds of Attacks</h1>
<p><b>Denial of Service Attack</b></p>
<p>DOS attack consists of overloading the device's resources, causing the unability to use it. It's almost always peformed from one or only a few devices. </p>
<p>Let's say you run a simple script that asks some queries in the search field of the website hundreds of times a second.  This causes increase in the load of the server's CPU and hard drive and it may become unaccesible. Simple example of DOS attack ;)</p>
<p><b>Distributed Denial of Service Attack</b></p>
<p>This time the attack is performed from not one, but hundreds of devices simultaneously aimed for one device. </p>
<p><b>Distributed Reflected Denial of Service Attack</b></p>
<p>More strengthened version of DDOS based on reflective amplification. It causes increase in bandwith of an attack without significant increase in the attackers resources. </p>
<p>To help you better understand DrDOS we will discuse one of the first ones - Smurf Attack. Let's take a look at this diagram provided by Cloudfare</p>

![smurf-attack](/img/ddos-introduction/smurf-attack.png){:class="img-responsive"}
<p>The idea behind the smurf attack is falsification of the packet's source address, which are then sent to the broadcast addresses. Attacker sends single packets (for example ICMP Echo Request) to the broadcast address, which resends the packets to all nodes in specified broadcast domain. </p>

<p> Essential thing to do for an attacker is to spoof the source IP address of a packet, so they would look like they were sent from the victim's machine. After broadcasting those ping packets to the devices in the broadcast domain, they all try to response to the source - which was spoofed into the machine of the victim.  </p>

<p>Another type of DrDOS is DNS DrDOS which uses the DNS protocol. Attacker sends packet with spoofed source address to the DNS servers - by sending a 64 byte packet, it can send the 512 byte packet as an answer - about 8 times bigger than it was!  </p>

<p>We can also divide the attacks by the layer that they're attacking. Let's look closer.</p>
<p><b>Layer 3/ Layer 4 DDOS</b></p>
<p>These attacks focus on the flooding the victim with packets or datagrams. They are one of the simplest attacks, yet very powerful if the number of attacking devices is very big. </p>
<p>One of the most interesting attacks was so called <b>Ping of Death</b>, which worked by sending to the victim an ICMP packet bigger than  65 535 bytes. Because of the bad implementation of TCP/IP in older operating systems like Microsoft Windows 95, this attack often crashed system trying to parse this packet. </p>
<p>Another type of attack may be <b>Fragmentation Attack</b> which uses the fact that bigger IP packets may be fragmented and victims machine has to connect them back together. Merging of packets uses some processor resources so it's a good addition to other attacks. In addition it allows to avoid the IDS/IPS system signature - one of the main layer of network security. </p>
<p><b>Layer 7 DDOS</b></p>
<p>In this kind of DDOS you don't attack the network infrastructure - but the networking application. It's main goal is to look for features in this application that causes bigger use in processor, hard drive etc. After finding one, attacker floods this vulnerabilities with queries (like HTTP POST) that causes maximized use in certain resource - for example processor's usage. Just like in our example with running script sending queries in the search field of the website - it overloads the CPU and hard drive which makes this website impossible to reach for other users.  </p>

<p>This attack is harder than Layer 3/ Layer 4 attacks, it needs more preparation but it's very dangerous and can cause a lot of harm. </p>

<h1>DDOS -How to Protect?</h1>

<p><b>Prevention</b></p>
<p>Firstly, it's essential to reduce the surface of an attack, mainly by detailed configuration of systems and networking devices.  It's possible by: </p>
<ol>
<li>Blocking the unwanted and invalid network traffic</li>
<li>Deactivation of IP broadcasting</li>
<li>Hardening of servers and networking devices</li>
<li>Splitting the services into many smaller ones (dedicated machine to each service)</li>
</ol>

<p><b>Detection</b></p>
<p>Another stage of DDOS protection is effective detection of one. Even if you think that it's not hard to detect large DDOS, attackers sometimes prepare reconnaissance to maximize their effectiveness - if you are able to detect this kind of activity, you may better prepare and resist one. There are a few steps that will help you detect and that way protect from DDOS. </p>

<ol>
<li>Implementation of dedicated firewall devices, honeypots, IDS/IPS and network monitoring. </li>
<li>Implementation of web application firewalls.</li>
<li>Detection of disturbance in network traffic (for example more packets with SYN flag than the ACK)</li>
</ol>

<p><b>Countermeasure</b></p>
<p>Often prevention and detection won't stop dedicated attacker that wants to keep your network down. Here are a few tips that may help you respond better to this attacks. </p>

<ol>
<li>Let specialized company deal with the DDOS. Sometimes companies like <a href="https://www.cloudflare.com/">Cloudfare</a> can deal with it better than you can on your own. </li>
<li>Localize vulnerable elements. Is the problem on the ISP's side or is it your server? Maybe search field on your website is generating to many queries? </li>
<li>Loadbalancing will allow you to distribute load between few machines which may reduce effects of attack. You can even use Global Server Load Balancing (GSLB) to reduce the effects even more. </li>
</ol>

<p>Of course there are more methods to respond to DDOS but you should choose appropriately to the specific incident. Also remember that some attacks may be impossible to block and only option to choose would be to wait it out. </p>

<p>I know it was only a brief overview of how DDOS attacks work, and if you're interested in reading more about them I can recommend you these resources: </p>
[OWASP - Denial of Service](https://www.owasp.org/index.php/Denial_of_Service "OWASP - Denial of Service")<br>
[DOS Attacks Mitigation](https://www.sans.org/reading-room/whitepapers/detection/denial-service-attacks-mitigation-techniques-real-time-implementation-detailed-analysi-33764 "DOS Attacks Mitigation")<br>
[Netflow Incident Detection](https://www.first.org/global/practices/Netflow.pdf "Netflow Incident Detection")<br>

<p>Thanks for exploring this topic with me, I hope you enjoyed and now understand how DDOS attacks work.  As they are getting more and more common it's good to know how they work and how to prevent them from happening. </p>

<p>~ Stay safe!</p>
