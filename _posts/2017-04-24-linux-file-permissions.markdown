---
layout:     post
title:      "File Permissions"
subtitle:   "Quick walkthrough"
date:       2017-04-24 0:00:00
author:     "W3ndige"
header-img: "img/file-permission-header.jpeg"
permalink: /:title/
category: "Linux"
---

<p>A little break in the exams, so we have some time to discover another topic - which is file permissions on Linux. Let's fully understand how they work. </p>

<hr>

<h1 id="permission-types">Permission Types</h1>

<p>There are three basic permission types: </p>

<ul>
  <li><b>read</b> - modifiy whether or not user can read the content of the file.</li>
  <li><b>write</b> - modifiy whether or not user can write or modify a file/directory.</li>
  <li><b>execute</b> - modifiy whether or not user can execute the file or view the content of the directory.</li>
</ul>

<p>Now we can add these permissions to the three types of people: </p>

<ul>
  <li><b>owner</b> - permissions apply only to the owner of the file or a directory.</li>
  <li><b>group</b> - permissions apply only to the group that has been assigned to the file or a directory.</li>
  <li><b>others</b> - permissions apply to every other user.</li>
</ul>

<h1>View Permission</h1>

<p>In order to view permissions we're going to use <b>ls -l</b> command. Let's take a look at a few examples. </p>

{% highlight bash %}
w3ndige@W3ndige ~/P/w/img> ls -l about-header.jpg
-rw-r--r-- 1 w3ndige users 107458 03-11 18:34 about-header.jpg
{% endhighlight %}

<p>And a directory. </p>

{% highlight bash %}
w3ndige@W3ndige ~/P/w3ndige.github.io> ls -l img
drwxr-xr-x 2 w3ndige users   4096 03-11 18:34 64base/
{% endhighlight %}

<p>As you can recon first character identifies, if that's the <b>file</b> (-) or a <b>directory</b> (d). Then we have 3 characters specified for a permissions to an <b>owner</b>, 3 to the <b>group</b> and lastly 3 for all the <b>other users</b>. Presence of permission is presented by a coresspondig letter, lack is shown as a dash. </p>

<h1>Change Permission</h1>

<p>In order to change permissions, we can use tool called <b>chmod</b> with this syntax <b>chmod PERM FILE</b>. Let's take a look at examples. But firstly, let's think to who are we granting with these permissions? Owner - <b>u</b>, group - <b>g</b>, others - <b>o</b>, or all - <b>a</b>. Are we giving these permissions - <b>+</b>, or revoking - <b>-</b>? And lastly, which permissions? </p>

{% highlight bash %}
w3ndige@W3ndige ~> touch example
w3ndige@W3ndige ~> ls -l example
-rw-r--r-- 1 w3ndige users 0 05-13 18:05 example
w3ndige@W3ndige ~> chmod u+x example
w3ndige@W3ndige ~> ls -l example
-rwxr--r-- 1 w3ndige users 0 05-13 18:05 example*
{% endhighlight %}

<p>Execute permission for owner? No problem. </p>

{% highlight bash %}
w3ndige@W3ndige ~> chmod o+w example
w3ndige@W3ndige ~> ls -l example
-rwxr--rw- 1 w3ndige users 0 05-13 18:05 example*
{% endhighlight %}

<p>Or write permission for others? Easy task. We can also assing multiple permissions at once just like this: </p>

{% highlight bash %}
w3ndige@W3ndige ~> chmod a+rwx example
w3ndige@W3ndige ~> ls -l example
-rwxrwxrwx 1 w3ndige users 0 05-13 18:05 example*
{% endhighlight %}

<h1>Numeric Permission</h1>

<p>In order to assign specific set of permissions faster, we can assign them as a numbers - in base 8. Firstly, we'll take a look at how to convert character permission into a numerical type. </p>

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th>Octal</th>
        <th>Binary</th>
      </tr>
    </thead>
    <tr>
      <th>0</th>
      <th>000</th>
    </tr>
    <tr>
      <th>1</th>
      <th>001</th>
    </tr>
    <tr>
      <th>2</th>
      <th>010</th>
    </tr>
    <tr>
      <th>3</th>
      <th>011</th>
    </tr>
    <tr>
      <th>4</th>
      <th>100</th>
    </tr>
    <tr>
      <th>5</th>
      <th>101</th>
    </tr>
    <tr>
      <th>6</th>
      <th>110</th>
    </tr>
    <tr>
      <th>7</th>
      <th>111</th>
    </tr>
  </table>
</div>

<p>Looking at that conversion table, we can notice a pattern - it is possible to write combinations of a permission in a single number - for example 7 will stand for read, write and execute, while number 5 will be only read and execute. Each bit in the binary represents the presence of permision. Now we have to only write this number 3 times, to represent each piece of users - owner, group and others. </p>

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th>Parameter</th>
        <th></th>
        <th>Description</th>
      </tr>
    </thead>
    <tr>
      <th>777</th>
      <th>rwxrwxrwx</th>
      <th>Full access</th>
    </tr>
    <tr>
      <th>755</th>
      <th>rwxr-xr-x</th>
      <th>Owner has full access, everyone else can read and execute the file</th>
    </tr>
    <tr>
      <th>700</th>
      <th>rwx------</th>
      <th>Only owner has full access</th>
    </tr>
    <tr>
      <th>644</th>
      <th>rw-r--r--</th>
      <th>Owner can read and write, everyone else can read</th>
    </tr>
    <tr>
      <th>600</th>
      <th>rw-------</th>
      <th>Owner can read and write, otherwise no access</th>
    </tr>
  </table>
</div>

<p>Now we are able to assign multiple permissions at the same time using <b>chmod</b>. Let's take a look at few examples. </p>

{% highlight bash %}
w3ndige@W3ndige ~> chmod 777 example
w3ndige@W3ndige ~> ls -l example
-rwxrwxrwx 1 w3ndige users 0 05-13 18:05 example*
{% endhighlight %}

<p>Full access to everyone. </p>

{% highlight bash %}
w3ndige@W3ndige ~> ls -l example
-rwxrwx--- 1 w3ndige users 0 05-13 18:05 example*
{% endhighlight %}

<p>After realizing what a mistake it is, we revoked full access from others. Easy, right? We can also take a look at some useful permissions for directories. </p>

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th>Parameter</th>
        <th></th>
        <th>Description</th>
      </tr>
    </thead>
    <tr>
      <th>777</th>
      <th>rwxrwxrwx</th>
      <th>Everyone is able to list files, create new ones and delete them</th>
    </tr>
    <tr>
      <th>755</th>
      <th>rwxr-xr-x</th>
      <th>Owner has full access, while all others can only list the directory </th>
    </tr>
    <tr>
      <th>700</th>
      <th>rwx------</th>
      <th>Only owner can do anything, others have no access </th>
    </tr>

  </table>
</div>


<h1>Changing ownership</h1>

<p>Usually, owner is the one who created the file, but we can change the person using <b>chown</b> command. We can also change the group ownership using <b>chgrp</b> command. Remember that in order to change these, you have to be the superuser. </p>

{% highlight bash %}
w3ndige@W3ndige ~> chown USER FILE
w3ndige@W3ndige ~> chgrp GROUP FILE
{% endhighlight%}

<p>File permissions are great way to improve security on your system, so it's great to know how they work, and how to apply them. See you in the next one! </p>

<p>~ Stay safe! </p>
