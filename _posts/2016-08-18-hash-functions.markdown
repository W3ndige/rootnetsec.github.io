---
layout:     post
title:      "Hash Functions"
subtitle:   "How passwords are stored?"
date:       2016-08-18 5:00:00
author:     "W3ndige"
header-img: "img/hash-functions.jpg"
permalink: /:title/
category: Cryptography
---
<h1>What Are Hash Functions?</h1>
<p>A hash function is a function that takes the input value, and from that input creates an output value different from the input. For any x input value, you will always receive the same y output value whenever the hash function is run. </p>
<p><i>f(x) = y</i></p>

<p>md5(somesecret) = 3e2a64a814b7d8db3a4832ceb6e43cff</p>

<p>This function is called md5 and it's one of many types of hashing functions. This one creates 32 character hexadecimal output. But why hashes are used and for what?</p>

<b>Story Time!</b>
<p>Bob wanted to test knowledge of Alice and asked her to complete some mathematical task. She finished it, and asks Bob for answer but since he doesn't know whether she's lying or not, he firstly hashes answer, and sends her this version with a note that she should also hash her answer and compare it to see if it's correct or not. That way he doesn't give her the possibility to cheat by looking into answer he sent her, and adjusting operations. </p>

<p>After simplyfing hashes are used in this context of verifying information without revealing it to the party that is verifying ;)</p>

<h1>Password Storage</h1>
<p>User information that you enter during the sign up (your username, password, age, any info you provide) are often stored in databases in a way that after login you are always provided with information you entered during the registration. However, if someone got unathorized access to the server (and to the database) they would gain all your credentials, which may end up pretty badly and you would be quite screwed up as they would try to login to different services like Amazon with same username and password you provided. </p>

<p>But instead of storing passwords in plaintext (unfortunately many companies still use this option :( ), developers came with process of hashing the password and storing it safely in the database. Now if you remember that every time that hash function gets input x it will produce same output y,for the string you enter as your password  it will always produce the same hash. That way during the login process hashed password stored in the database can be compared with the hashed string you entered in the password field and if they're the same, then you are given an access to  your account. </p>

![hash-login](/img/hash-functions/hash-login.png){:class="img-responsive"}

<b>One more story time!</b>
<p>Bob just created his new website allowing people to connect and learn together. Alice wanted to try it out so she created new account with username: alice and password: mysecretpassword. The website created new database entry for that account, hashes the password and now she can login any time to learn something new. Take a look at the database: </p>
<p>bob:99700f3a1f08928115382fe09f850340</p>
<p>eve:b8ac4414ed78205c46eabb63d646f07f</p>
<p>gabe:5f4dcc3b5aa765d61d8327deb882cf99</p>
<p>alice:4cab2a2db6a3c31b01d804def28276e6</p>
<p>Two days later Alice logins once again, she enters her username "alice" and password which she mispelled "mysecretpasswor". Website generates a hash from the string in the password field (md5(mysecretpasswor) = eb61c2c9887a49a12a3bc3737aa7bc49) and compares it with the password for user "alice" in the database. It doesn't match so website shows error that password was not correct. Alice tries once again, this time correctly with matching password and she has gained the access. Also notice how different the hashed version is even by mispelling one letter. This is due to what is called the <b>avalanche effect</b>, where even small differences in the input will result in vastly different outputs.</p>
<p>Hash functions are designed to be very difficult to reverse – that way no one can view our passwords. But what if someone actually tries to attack hashes? He can try something called <b>rainbow tables</b></p>

<h1>Rainbow Tables</h1>
<p>But what is a rainbow table and how can it help an attacker? Rainbow table is a table that contains precomputed hashes for words, leaked passwords etc. It can look somehow like this: </p>

<table>
<tr>
<th>ID</th>
<th>Word</th>
<th>Hash</th>
</tr>
<tr>
<th>1</th>
<th>hello</th>
<th>5d41402abc4b2a76b9719d911017c592</th>
</tr>
<tr>
<th>2</th>
<th>password</th>
<th>5f4dcc3b5aa765d61d8327deb882cf99</th>
</tr>
<tr>
<th>3</th>
<th>password123</th>
<th>482c811da5d5b4bc6d497ffa98491e38</th>
</tr>
<tr>
<th>4</th>
<th>youcantseeme</th>
<th>ffe60ecaa8bba2f12b43d1a4b15b8f39</th>
</tr>
<tr>
<th>5</th>
<th>verysafepassword</th>
<th>62edefe87740267ee613d704588a73a7</th>
</tr>

</table>

<p>Now the attacker doesn't have to compute all the hashes, he "just" have to compare the entry from the database with every password in the rainbow table. Notice how in the previous story the password for gabe account is same as hash in the rainbow table for the word "password" ? That's how the attacker gained knowledge of gabe's password - now he can try it on every other gabe account like Amazon.</p>

<p>A large portion of the security of the hash (the fact that the hash function is not an instant operation, and requires some amount of computational power, and therefore time, to perform) is bypassed with rainbow tables, meaning that if you’re lucky, you’ll find some matches.</p>

<p>Unfortunately, rainbow tables have some disadvantages - there is no rainbow table with all possible combinations of words, letters, characters or numbers so if the password you're looking for is not there - you'll have to try and brute force it. Also even though you do not have to perform all computation, their size is huge and you now have to store significant amount of data just to crack passwords. Additionaly, as the popularity of less secure hashing algorithms fell, and as password salting became a more common practice, rainbow tables have fallen out of common use.</p>

<h1>Password salting</h1>
<p>Salting is adding unique data (like random string, username or registration date) to the password before process of hashing. It was created to prevent cracking password with rainbow tables as attacker now have to get salt and a password.  </p>
<p>md5(password) = 5f4dcc3b5aa765d61d8327deb882cf99</p>
<p>md5(password + alice) = dc9478eb4e94a7dcf2bda02360188a52</p>
<p>Different than original password, right?</p>
<p>Now, if someone were to try the same rainbow table attack with a list of common password hashes – none of the hashes would match. The salts change the output of the hash function completely, so even the common passwords are safe. If an attacker wanted to crack the passwords now, they would have to add the salt for each individual user – increasing the amount of time required to brute force all of the passwords exponentially.</p>

<h1>Hash Collision Attack</h1>
<p>Hash collision occurs when 2 different inputs produce the same output (hash). ecause hash functions have infinite input length and a predefined output length, there is inevitably going to be the possibility of two different inputs that produce the same output hash. Of course odds of collision are very low, but md5 and even SHA-1 have been shown to not be very collision resistant – however stronger functions such as SHA-256 seem to be safe at the current time.</p>
<p>Here there are two binaries with the same md5 hash sum:</p>
{% highlight text %}
$ md5sum message1.bin message2.bin
008ee33a9d58b51cfeb425b0959121c9  message1.bin
008ee33a9d58b51cfeb425b0959121c9  message2.bin
{% endhighlight %}
[Read more](https://marc-stevens.nl/research/md5-1block-collision/ "Read More")

<h1>But Why Hashes Are Irreversible?</h1>
<p>It may be hard to understand why hash function is an one way cryptographic function but there is very good answer to that - modulo operator. For a quick review, modulus is essentially the same as saying “the remainder of” (applying to division).</p>
<p>15 mod 5 = 0 </p>
<p><i>As 15 / 5 = 3 without the remainder</i></p>
<p>24 mod 7 = 3</p>
<p><i>As 15 / 7 = 3 with remainder 3</i></p>
<p>But why modulo?</p>
<p>Basically because it's irreversible, we know the result but basically there are so many combinations that it's simply impossible to choose the correct one. For example we know that the result is 3 but both 24 mod 7 = 3 and 18 mod 5 = 3 etc. Additionally a lot of from the input is discarded due to the limited size of the hash. It would be impossible to figure out the original data of the function with just the resulting hash – as not much of that data is left. If we could reverse a hash, we would be able to compress data of any size into a few bytes of data!</p>
<hr>
<p>I hope that now you understand how hash functions work, what is salt and why hashes are irreversible. Tune more for the comparision between modern hashing algorithms. And remember, always salt your passwords!</p>
<p>~Stay safe!</p>
