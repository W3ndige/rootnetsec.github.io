---
layout:     post
title:      "Nmap"
subtitle:   "Cheat Sheet"
date:       2017-02-25 00:00:00
author:     "W3ndige"
header-img: "img/nmap-header.jpg"
category: "Cheat Sheets"
---

<h1>What is Nmap?</h1>

<p><b>Nmap</b>, short for network mapper, is an open source security tool for network exploration, security scanning and auditing. It was originally written by Gordon Lyon AKA Fyodor, firstly published in September 1997 in a <a href="http://phrack.org/issues/51/11.html">Phrack Magazine</a>, with included source code. </p>

<p>It uses raw IP packets in novel ways to determine what hosts are available on the network, what services those hosts are running, OS detection, what type of packet filters/firewalls are in use, and many others. </p>


<p>Nmap has even made appearance in a few movies, including <b>The Matrix Reloaded</b>, where Trinity needing to hack the city power grid, she whips out Nmap, uses it to find a vulnerable SSH server, and then proceeds to exploit it using the SSH1 CRC32 exploit from 2001. Of course there are more films, such as <b>The Bourne Ultimatum</b>, <b>Girl With the Dragon Tattoo</b>, <b>Live Free or Die Hard</b>, or even computer game called <b>Hacknet</b>. </p>

![Matrix-Reloaded](/img/nmap/nmap-matrix.jpg){:class="img-responsive center-block"}

<h1>Table of Contents</h1>
<ul>
  <li><h3><a href="#fast-start">Fast Start</a></h3></li>
  <li><h3><a href="#target-selection">Target Selection</a></h3></li>
  <li><h3><a href="#port-selection">Port Selection</a></h3></li>
  <li><h3><a href="#probing-options">Probing Options</a></h3></li>
  <li><h3><a href="#timing-options">Timing Options</a></h3></li>
  <li><h3><a href="#scan-techniques">Scan Techniques</a></h3></li>
  <li><h3><a href="#service-version-detection">Service Version Detection</a></h3></li>
  <li><h3><a href="#os-detection">OS Detection</a></h3></li>
  <li><h3><a href="#firewall-evasion">Firewalls IDS Evasion and Spoofing</a></h3></li>
  <li><h3><a href="#scripting">Scripting</a></h3></li>
  <li><h3><a href="#output-options">Output Options</a></h3></li>
  <li><h3><a href="#miscellaneous">Miscellaneous</a></h3></li>
</ul>
<h1 id="fast-start">Fast Start</h1>

<p>Firstly I'll discuss a few commands, that are essential for me.  </p>

<b>1. nmap -sP 192.168.0.1/24</b>
<p>This command, also known as "ping scan", tells Nmap to send an <b>ICMP echo</b> request, <b>TCP SYN</b> to port 443, <b>TCP ACK</b> to port 80 and <b>ICMP timestamp</b> request to all hosts in the specified subnet. Then, Nmap will return a list of IP's that have responded. </p>

<p>This command <b>does not require root</b>, however while used with root account, Nmap will send additional <b>ARP</b> requests. </p>

<b>2. nmap 192.168.0.1/24</b>

<p>Default scan, that will look for open ports. Nmap will try and attempt a <b>TCP SYN</b> connection to 1000 most common ports, as well as an <b>ICMP echo</b> request to determine if a host is up. In addition, <b>DNS lookup</b> will be performed, possibly giving us some useful information. <b>No root required</b>.  </p>

<b>3. nmap -T4 -F 192.168.0.100</b>

<p>Fast scan, that will look at 100 most common ports. <b>No root required</b>.</p>

<b>4. nmap -sS -sU -Pn -p- 192.168.0.100</b>

<p><b>TCP SYN</b> and <b>UDP</b> scan is rather unobtrusive and stealthy. <b>-Pn</b> flag will also skip the ping scan, assuming that all hosts are up (very useful when there is a firewall preventing ICMP replies). In addition <b>-p-</b> will scan all 65535 ports. </p>

<b>5. nmap -v -sS -A -T4 192.168.0.100</b>

<p>This command will run <b>TCP SYN</b> scan, with <b>OS and version detection</b> + <b>traceroute</b>, <b>T4</b> timing and, in addition, will print <b>verbose output</b>. </p>

<h1 id="target-selection">Target Selection</h1>

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th>Parameter</th>
        <th>Description</th>
      </tr>
    </thead>
    <tr>
      <th>192.168.0.1</th>
      <th>Scan a single IPV4</th>
    </tr>
    <tr>
      <th>AABB:CCDD::FF%eth0</th>
      <th>Scan a single IPV6</th>
    </tr>
    <tr>
      <th>192.168.0.1-15</th>
      <th>Scan a range of IP's</th>
    </tr>
    <tr>
      <th>192.168.0.1/24</th>
      <th>Scan a subnet</th>
    </tr>
    <tr>
      <th>www.hostname.com</th>
      <th>Scan a hostname</th>
    </tr>
    <tr>
      <th><span class="text-danger">-iL</span> list.txt</th>
      <th>Scan targets from text file</th>
    </tr>
  </table>
</div>

<h1 id="port-selection">Port Selection</h1>

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th>Parameter</th>
        <th>Description</th>
      </tr>
    </thead>
    <tr>
      <th><span class="text-danger">-p</span> <span class="text-success">22</span> 192.168.0.1</th>
      <th>Scan a single port</th>
    </tr>
    <tr>
      <th><span class="text-danger">-p</span> <span class="text-success">1-100</span> 192.168.0.1</th>
      <th>Scan a range of ports</th>
    </tr>
    <tr>
      <th><span class="text-danger">-F</span> 192.168.0.1</th>
      <th>Scan 100 most common ports</th>
    </tr>
    <tr>
      <th><span class="text-danger">-p-</span> 192.168.0.1</th>
      <th>Scan all 65535 ports</th>
    </tr>
    <tr>
      <th><span class="text-danger">-p</span> <span class="text-success">U:PORT</span> 192.168.0.1</th>
      <th>Scan UDP ports, ex U:53,U:110</th>
    </tr>
    <tr>
      <th><span class="text-danger">-r</span> 192.168.0.1</th>
      <th>Do not randomize ports</th>
    </tr>
  </table>
</div>

<h1 id="probing-options">Probing Options</h1>

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th>Parameter</th>
        <th>Description</th>
      </tr>
    </thead>
    <tr>
      <th><span class="text-danger">-pN</span></th>
      <th>Assume all hosts are up</th>
    </tr>
    <tr>
      <th><span class="text-danger">-PB</span></th>
      <th>Default probe</th>
    </tr>
    <tr>
      <th><span class="text-danger">-PS/PA/PU/PY</span>[portlist]</th>
      <th>TCP SYN/ACK, UDP or SCTP ports probing</th>
    </tr>
    <tr>
      <th><span class="text-danger">-PE/PP/PM</span></th>
      <th>ICMP Echo, Timestamp, and Netmask probing</th>
    </tr>
  </table>
</div>


<h1 id="timing-options">Timing Options</h1>

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th>Parameter</th>
        <th>Description</th>
      </tr>
    </thead>
    <tr>
      <th><span class="text-danger">-T0</span></th>
      <th><i>Paranoid</i> Very slow, used for IDS evasion</th>
    </tr>
    <tr>
      <th><span class="text-danger">-T1</span></th>
      <th><i>Sneaky</i> Quite slow, used for IDS evasion</th>
    </tr>
    <tr>
      <th><span class="text-danger">-T2</span></th>
      <th><i>Polite</i> Runs around 10 times slower than normal, used to save bandwidth</th>
    </tr>
    <tr>
      <th><span class="text-danger">-T3</span></th>
      <th><i>Normal</i> A dynamic timing model based on target responsiveness</th>
    </tr>
    <tr>
      <th><span class="text-danger">-T4</span></th>
      <th><i>Aggresive</i> Assumes a fast and reliable network</th>
    </tr>
    <tr>
      <th><span class="text-danger">-T5</span></th>
      <th><i>Insane</i> Fastest option, will likely overwhelm targets or miss open ports</th>
    </tr>
    <tr>
      <th><span class="text-danger">--scan-delay</span><br><span class="text-danger">--max-scan-delay</span> TIME</th>
      <th>Adjust delay between probes</th>
    </tr>
    <tr>
      <th><span class="text-danger">--host-timeout</span> TIME</th>
      <th>Give up on target after specified time</th>
    </tr>
    <tr>
      <th><span class="text-danger">--min-hostgroup</span><br><span class="text-danger">--max-hostgroup</span> TIME</th>
      <th>Parallel host scan group sizes</th>
    </tr>
    <tr>
      <th><span class="text-danger">--min-parallelism</span><br><span class="text-danger">--max-parallelism </span> TIME</th>
      <th>Probe parallelization</th>
    </tr>
    <tr>
      <th><span class="text-danger">--min-rtt-timeout</span><br><span class="text-danger">--max-rtt-timeout </span><br><span class="text-danger">--initial-rtt-timeout </span> TIME</th>
      <th>Specifies probe round trip time</th>
    </tr>
    <tr>
      <th><span class="text-danger">--max-retries</span> TRIES</th>
      <th>Caps number of port scan probe retransmissions</th>
    </tr>
    <tr>
      <th><span class="text-danger">--min-rate</span> NUMBER</th>
      <th>Send packets no slower than NUMBER per second</th>
    </tr>
    <tr>
      <th><span class="text-danger">--max-rate</span> NUMBER</th>
      <th>Send packets no faster than NUMBER per second</th>
    </tr>

  </table>
</div>

<h1 id="scan-techniques">Scan Techniques</h1>

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th>Parameter</th>
        <th>Description</th>
      </tr>
    </thead>
    <tr>
      <th><span class="text-danger">-Ss</span> </th>
      <th>TCP SYN Scan</th>
    </tr>
    <tr>
      <th><span class="text-danger">-sN</span> </th>
      <th>TCP NULL Scan</th>
    </tr>
    <tr>
      <th><span class="text-danger">-sU</span> </th>
      <th>UDP Scan</th>
    </tr>
    <tr>
      <th><span class="text-danger">-sF</span> </th>
      <th>Fin Scan</th>
    </tr>
    <tr>
      <th><span class="text-danger">-sF</span> </th>
      <th>ACK Scan</th>
    </tr>
    <tr>
      <th><span class="text-danger">-sW</span> </th>
      <th>Windows Scan</th>
    </tr>
    <tr>
      <th><span class="text-danger">-sT</span> </th>
      <th>Connect Scan</th>
    </tr>
    <tr>
      <th><span class="text-danger">-sX</span> </th>
      <th>Xmas Scan</th>
    </tr>
    <tr>
      <th><span class="text-danger">-sM</span> </th>
      <th>Maimon Scan</th>
    </tr>
  </table>
</div>

<h1 id="service-version-detection">Service Version Detection</h1>

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th>Parameter</th>
        <th>Description</th>
      </tr>
    </thead>
    <tr>
      <th><span class="text-danger">-sV</span> </th>
      <th>Version Scan</th>
    </tr>
    <tr>
      <th><span class="text-danger">--version-intensity "intensity"</span> </th>
      <th>Set intensity from 0 (light) to 9 (try all probes)</th>
    </tr>
    <tr>
      <th><span class="text-danger">--version-light</span> </th>
      <th>Intensity 2</th>
    </tr>
    <tr>
      <th><span class="text-danger">--version-all</span> </th>
      <th>Intensity 9</th>
    </tr>
    <tr>
      <th><span class="text-danger">--version-trace</span> </th>
      <th>Show detailed version scan activity</th>
    </tr>
  </table>
</div>

<h1 id="os-detection">OS Detection</h1>

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th>Parameter</th>
        <th>Description</th>
      </tr>
    </thead>
    <tr>
      <th><span class="text-danger">-O</span></th>
      <th>Enable OS Detection</th>
    </tr>
    <tr>
      <th><span class="text-danger">--osscan-limit</span></th>
      <th>Limit OS detection to only promising targets</th>
    </tr>
    <tr>
      <th><span class="text-danger">--osscan-guess</span></th>
      <th>Guess OS more aggressively</th>
    </tr>

  </table>
</div>

<h1 id="firewall-evasion">Firewalls IDS Evasion and Spoofing</h1>

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th>Parameter</th>
        <th>Description</th>
      </tr>
    </thead>
    <tr>
      <th><span class="text-danger">-S</span> IP</th>
      <th>Spoof IP address</th>
    </tr>
    <tr>
      <th><span class="text-danger">--spoof-mac</span> MAC</th>
      <th>Spoof MAC address</th>
    </tr>
    <tr>
      <th><span class="text-danger">--data-length</span> NUM</th>
      <th>Append random data to packets</th>
    </tr>
    <tr>
      <th><span class="text-danger">-g</span> PORT<br><span class="text-danger">--source-port</span> PORT</th>
      <th>Use given port number</th>
    </tr>
    <tr>
      <th><span class="text-danger">-D </span> decoy1,decoy2,ME</th>
      <th>Cloak a scan with decoys</th>
    </tr>
    <tr>
      <th><span class="text-danger">--badsum </span></th>
      <th>Send packets with a fake TCP/UDP/SCTP checksum</th>
    </tr>
    <tr>
      <th><span class="text-danger">--ttl </span>VALUE</th>
      <th>Set IP time to live field</th>
    </tr>

  </table>
</div>

<h1 id="scripting">Nmap Scripting</h1>

<p>Firstly, let's discuss different categories of scripting in Nmap. </p>

<p><b>Auth</b> - utilize credentials or bypass authentication on target
hosts.</p>

<p><b>broadcast</b> - discover hosts not includedon command line by broadcasting on local network. </p>

<p><b>brute</b> - attempt to guess passwords on target systems, for avariety of protocols, including http, SNMP, IAX, MySQL, VNC,etc.</p>

<p><b>default</b> - scripts run automatically when -sC or-A are used.</p>

<p><b>discovery</b> - try to learn more information about target hosts through public sources of information, SNMP, directory services, and more. </p>

<p><b>dos</b> - may cause denial of service conditions in target hosts. </p>

<p><b>exploit</b> - attempt to exploit target systems. </p>

<p><b>external</b> - interact with third partysystems not included intarget list. </p>

<p><b>fuzzer</b> - send unexpected input in network protocol fields.</p>

<p><b>intrusive</b> - may crash target, consume excessive resources, or otherwise impact target machines in a malicious fashion. </p>

<p><b>malware</b> - look for signs of malware infection on the target hosts. </p>

<p><b>safe</b> - designed not to impact target in a negative fashion. </p>

<p><b>version</b> - measure the version of software or protocol spoken by target hosts. </p>

<p><b>vul</b> - measure whether target systems have a known vulnerability </p>

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th>Parameter</th>
        <th>Description</th>
      </tr>
    </thead>
    <tr>
      <th><span class="text-danger">-sC</span></th>
      <th>Run default scripts</th>
    </tr>
    <tr>
      <th><span class="text-danger">--script</span>="category"</th>
      <th>Enter here script category or script-file</th>
    </tr>
    <tr>
      <th><span class="text-danger">--script-updatedb</span></th>
      <th>Update script database</th>
    </tr>
    <tr>
      <th><span class="text-danger">--script-help</span>="category"</th>
      <th>Show help for category</th>
    </tr>
  </table>
</div>

<h1 id="output-options">Output Options</h1>

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th>Parameter</th>
        <th>Description</th>
      </tr>
    </thead>
    <tr>
      <th><span class="text-danger">-oN</span></th>
      <th>Standard Nmap output</th>
    </tr>
    <tr>
      <th><span class="text-danger">-oG</span></th>
      <th>Greppable format</th>
    </tr>
    <tr>
      <th><span class="text-danger">-oX</span></th>
      <th>XML format</th>
    </tr>
    <tr>
      <th><span class="text-danger">-oA</span> NAME</th>
      <th>Generate Nmap, greppable, and XML output files using name for files</th>
    </tr>


  </table>
</div>

<h1 id="miscellaneous">Miscellaneous</h1>

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th>Parameter</th>
        <th>Description</th>
      </tr>
    </thead>
    <tr>
      <th><span class="text-danger">-n</span></th>
      <th>Disable reverse IP address lookups</th>
    </tr>
    <tr>
      <th><span class="text-danger">-6</span></th>
      <th>Use IPv6 only</th>
    </tr>
    <tr>
      <th><span class="text-danger">-A</span></th>
      <th>Use scan types including OS detection, version detection, script scanning in default category, and traceroute</th>
    </tr>
    <tr>
      <th><span class="text-danger">--reason</span></th>
      <th>Display reason Nmap thinks port is open, closed, or filtered</th>
    </tr>
  </table>
</div>

<h1>Last words and references</h1>

<p>Firstly, I can recommend this great piece of information <a href="https://nmap.org/book/toc.html">Nmap Network Scanning Digital Version</a>, which covers a lot of topics in this amazing tool. In addition I will keep this cheat sheet as update as possible, with new content coming as soon as I'll learn more, so stay updated! And...</p>

<p>~ Stay safe!</p>
