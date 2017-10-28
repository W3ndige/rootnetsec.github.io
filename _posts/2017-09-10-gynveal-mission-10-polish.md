---
layout:     post
title:      "Gynvael Polish Mission 010"
subtitle:   "Write-Ups"
date:       2017-09-10 8:00:00
author:     "W3ndige"
permalink: /:title/
category: Write-Ups
---

<p>Let's get back to work, and as Gynvael started streaming again, his mission should be great start. </p>

<pre>
MISSION 010            goo.gl/oAdvWe                  DIFFICULTY: ███░░░░░░░ [3/10]
┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅

We've received PDF document, where apparently there's a hidden message.
Would you help us find it?

  https://goo.gl/wgt94W

Good luck!
</pre>

<p>I started this challenge by looking at the file with a hex editor. </p>

![Hexdump](/img/gynvael-missions/pdf_hex_editor.png){:class="img-responsive center-block"}

<p>After a few moments of searching, I've found the <b>JFIF</b> segment and so, the whole structure of <b>JPG</b> file. From that moment, I think we'll be able to extract it from the PDF and somehow recreate it. </p>

<p>Quick peek at <a href="https://en.wikipedia.org/wiki/JPEG_File_Interchange_Format">Wikipedia</a> showed that whole JPG file is placed between <code>FF DA</code> and <code>FF D9</code> codes. </p>

<table><tr>
<th colspan="3" style="text-align:left">JFIF file structure</th>
</tr>
<tr>
<th style="text-align:left">Segment</th>
<th style="text-align:left">Code</th>
<th style="text-align:left">Description</th>
</tr>
<tr>
<td>SOI</td>
<td><code>FF D8</code></td>
<td>Start of Image</td>
</tr>
<tr>
<td>JFIF-APP0</td>
<td><code>FF E0 <i>s1</i> <i>s2</i> 4A 46 49 46 00 ...</code></td>
<td>see below</td>
</tr>
<tr>
<td>JFXX-APP0</td>
<td><code>FF E0 <i>s1</i> <i>s2</i> 4A 46 58 58 00 ...</code></td>
<td>optional, see below</td>
</tr>
<tr>
<td colspan="3">… additional marker segments<br>
(for example SOF, DHT, COM)</td>
</tr>
<tr>
<td>SOS</td>
<td><code>FF DA</code></td>
<td>Start of Scan</td>
</tr>
<tr>
<td></td>
<td>compressed image data</td>
<td></td>
</tr>
<tr>
<td>EOI</td>
<td><code>FF D9</code></td>
<td>End of Image</td>
</tr>
</table>

<p>Now let's copy everything between these segments and add to a new file. </p>

![hex image](/img/gynvael-missions/hex_image.png){:class="img-responsive center-block"}

<p>Last step is to save this file as JPG image. By the way, here's the flag ;)</p>

![flag](/img/gynvael-missions/pdf_flag.jpg){:class="img-responsive center-block"}
