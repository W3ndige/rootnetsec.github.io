---
layout:     post
title:      "Wireless Security"
subtitle:   "Theory behind WPA and WPA2"
date:       2016-10-01 2:00:00
author:     "W3ndige"
permalink: /:title/
category: Networking
---

<p>Hi everyone! Today we're gonna prepare a little bit before the next episode - in which we'll try to hack WPA and WPA Wi-Fi networks. But in order to do that we have to gain better understanding how it works, it's weaknesses (if any) and stronger sides that we shouldn't even try atttacking. Let's jump in to the world of theory! </p>

<h1>What is WPA? </h1>

<p>Due to the lack of security in WEP standard, Wi-Fi Alliance started working on the new standard known as 802.11i. But because the   requirements of this new standard were too high for some devices, in 2003 WPA standard was introduced as a temporary soluton - fast upgrade was possible because of the simple software upgrade. WEP included some of the security features dedicated for 802.11i - called WPA2.  </p>

<p>Main problems that 802.11i wanted to solve</p>

<ul>
<li><b>Protection of shared key</b></li>
<li><b>Security of encryption</b></li>
<li><b>Authentication</b></li>
<li><b>Variability of the key</b></li>
<li><b>Integity</b></li>
</ul>

<h1>RSN</h1>

<p>In order to rebuild the concept of security in wireless networks 802.11i standard introduced <b>Robust Security Network</b>.</p>

<p>In WPA there are 2 possible ways of authentication: </p>

<ul>
<li>Using <b>Pre-Shared-key</b> (PSK) known to the everyone in the network</li>
<li>Using more advanced credentials like own username and password or <b>X.509 certificates</b>. </li>
</ul>

<p>Access to the Wi-Fi using PSK works just like in the WEP, user before connecting to the network must enter a key. This solution, due to the simplicity, is used in homes and is called <b>WPA-Personal</b>, or WPA-PSKA. PSK is entered as 64 hexadecimal numbers, or 8-63 ASCII character.  </p>

<p>Second solution called <b>WPA-Enterprise</b> is using another element - Radius server carrying out the process of authentication, authorization and accounting. That way we gain a few more possibilities: </p>

<ul>
<li>Unique credentials for each user</li>
<li>Possibility to authenticate network to which user is connected (no more fake access point!)</li>
<li>Authentication proceeds before connecting to the network</li>
<li>Ability to control user on the network (time limit, connection to specific VLAN, etc)</li>

</ul>

<p>Firstly, when the user is trying to connect to the network, access point connects to the <b>Radius server</b>, sending him credentials passed by the user. Then Radius decides whether to allow the user to connect or not, and sends this decision to AP. Lastly AP fulfills decision of the server. </p>

<p><b>Important!</b> You have to remember that WPA-Enterprise should not be confused with <b>captive portals</b>, that require user to enter his credentials on WWW website in order to gain access to the Internet. In WPA-Enterprise authentication occurs before connecting the user to the network, while in captive portals it occurs after connection.  </p>

<p>In 802.11i it's essential to create two pairs of keys used to encrypting the traffic - for individual broadcast it's <b>PTK (Pairwise Transient Key)</b> and for group broadcast <b>GTK (Group Transient Key)</b>. Group key must be refreshed every time device disconnects from the network, in order to make impossible to get multicast or broadcast packets.  </p>

<p>In order to create those keys, there was introduced <b>4 way handshake operation</b>, used to create PTK and GTK keys with the PMK.  </p>

![4-way-handshake](/img/wireless-security-wpa/4-way-handshake.png){:class="img-responsive"}

<ol>

    <li>The AP sends a nonce-value to the STA (ANonce). The client now has all the attributes to construct the PTK.</li>
    <li>The STA sends its own nonce-value (SNonce) to the AP together with a MIC, including authentication, which is really a Message Authentication and Integrity Code (MAIC).</li>
    <li>The AP constructs and sends the GTK and a sequence number together with another MIC. This sequence number will be used in the next multicast or broadcast frame, so that the receiving STA can perform basic replay detection.</li>
    <li>The STA sends a confirmation to the AP.</li>

</ol>

<p>Whenever there will be need to create new GTK key, after the disconnection of the client, access point sends new key and connected client only accepts this change.  </p>

<p>After 4 way handshake client gets PTK and GTK keys, that are essential to encrypt the data. They are constant for each session (if there wasn't key renegotiation). To make it work 802.11i standard introduced CCMP (for WPA it was TKIP which made possible faster implementation). Learn more at:  </p>

[TKIP](https://en.wikipedia.org/wiki/Temporal_Key_Integrity_Protocol "TKIP")<br>
[CCMP](https://en.wikipedia.org/wiki/CCMP "CCMP")<br>


<h1>Security</h1>

<ul>
<li>Only eavesdropping the broadcast won't allow us to decrypt it due to the changes in encryption key (while in WEP you only had to capture big number of packets to crack the password). </li>

<li>Possesion of captured broadcast without the handshake won't let us decrypt it. </li>

<li>Fix in the IV collision</li>

<li>Use of algorithm insusceptible to the related key attack. </li>

<li>Better integrity of the packets</li>

<li>More advanced authentication mechanisms using 802.1X.</li>
</ul>
<h1>But is WPA vulnerable?</h1>

<p>Yes, and here are the most important types of attacks. </p>

<h3>Brute force attacks</h3>

<p>Due to the fact that PTK key, used to protect the broadcast is created using PSK and some data sent through handshake. </p>

<p>Even if we don't know the PSK, but we can capture additional data used during the handshake, we can still try to perform a brute force attack based on trying different PSK values, then calculating PMK and PTK and checking if the value is correct</p>

<p>But because of the size of PSK's value (9-63 ASCII characters) trying out whole number of combinations is almost impossible. In real life this attack is performed using dictionary attack which is a lot faster since trying out all the words in dictionary can be done in a few days, or even faster. There is only one flaw in this attack - if the password isn't in the dictionary - then we will not succeed. </p>

<p>Alternative to this, we can try attacking using rainbow tables. Because hashing algorithms create hashes of same and always equal size it's very hard to avoid phenomenon called collision. Collision occurs when two different inputs passed to the hashing algorithms produce same output. It makes the process of attacking even faster as we're using precomputed rainbow tables (not hashing in 'air'), and we only have to check whether each hash in the table works. Still we don't have certainity to succeed.  </p>

<p>Additionaly, in 2007, new vulnerability was described called "short packed spoofing", allowing us to decrypt short packets like ARP, but it didn't result in capturing the PSK key - only the key stream used to encrypt the packet, allowing us to inject few packets into the network, resulting in for example sending data from the user to the Internet.  </p>

<h3>Fuzzing</h3>

<p>Security of the wireless networks also depends on the software handling this work (drivers, libraries etc). Because of the fact that network architecture is getting more and more complicated - code is also getting much more complex which may result in flaws that attacker can use to exploit. </p>

<hr>
<p>I hope that you gained better understanding of how WPA and WPA2 works. In next part we're going to actually try and attack protected wireless network.  </p>
