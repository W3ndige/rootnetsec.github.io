---
layout:     post
title:      'Midnightsun CTF 2019 - Marcodowno'
date:       2019-04-06 0:00:00
author:     'W3ndige'
permalink: /:title/
category: 'Midnightsun CTF 2019'
---

This challenge I decided to go for a cool challenge, `marcodowno`, in which we had to find XSS vulnerability that pops `alert(1)` without the user interaction. After successfuly doing it in our client, we had to paste the working solution in the form of `URL` into a service that would check it, and upon correct exploitation, grant us a flag. 

Upon visiting a website, we can see a service that converts `markdown` input into a `html` code and views it. 

![Marcodowno-website](/img/midnightsun/marcodowno-website.png)

If we view the source of the website, the main part that converts the page is a simple `regexp` one liner searching for `markdown` components and replacing them with `html`. 

```javascript
input = decodeURIComponent(location.search.match(/input=([^&#]+)/)[1]);

function markdown(text){
  text = text.replace(/[<]/g, '').replace(/----/g,'<hr>').replace(/> ?([^\n]+)/g, '<blockquote>$1</blockquote>').replace(/\*\*([^*]+)\*\*/g, '<b>$1</b>').replace(/__([^_]+)__/g, '<b>$1</b>').replace(/\*([^\s][^*]+)\*/g, '<i>$1</i>').replace(/\* ([^*]+)/g, '<li>$1</li>').replace(/##### ([^#\n]+)/g, '<h5>$1</h5>').replace(/#### ([^#\n]+)/g, '<h4>$1</h4>').replace(/### ([^#\n]+)/g, '<h3>$1</h3>').replace(/## ([^#\n]+)/g, '<h2>$1</h2>').replace(/# ([^#\n]+)/g, '<h1>$1</h1>').replace(/(?<!\()(https?:\/\/[a-zA-Z0-9./?#-]+)/g, '<a href="$1">$1</a>').replace(/!\[([^\]]+)\]\((https?:\/\/[a-zA-Z0-9./?#]+)\)/g, '<img src="$2" alt="$1"/>').replace(/(?<!!)\[([^\]]+)\]\((https?:\/\/[a-zA-Z0-9./?#-]+)\)/g, '<a href="$2">$1</a>').replace(/`([^`]+)`/g, '<code>$1</code>').replace(/```([^`]+)```/g, '<code>$1</code>').replace(/\n/g, "<br>");
  return text;
}

window.onload=function(){
  $("#markdown").text(input);
  $("#rendered").html(markdown(input));
}
```

I immedietally went for something which can spawn the XSS without the user interaction, which is `img` tag. Let's take a look at the part converting `[]()` notation into a `<img>`.

```javascript
.replace(/!\[([^\]]+)\]\((https?:\/\/[a-zA-Z0-9./?#]+)\)/g, '<img src="$2" alt="$1"/>')
```

As we can see, we can put any characters in the first brackets, while in the source brackets we have to have something starting with `https://`. We can check that with [this](https://regexr.com/).

![Marcodowno-regexp](/img/midnightsun/marcodowno-regexp.png)

After regexp finds an exact match, it will place the content of the second brackets just where `$2` is, while content of first brackets inside `$1` in the `img src="$2" alt="$1"/>`. As you can probably see already, we can put inside the second brackets `"` character to break out of `alt`, the use `onerror=alert(1)` to spawn the XSS.

```markdown
!["onerror="alert(1)](https://asd.asd)
```

![Marcodowno-xss](/img/midnightsun/marcodowno-xss.png)

After that we can submit the `URL` into a service and get the flag. 

```
midnight{wh0_n33ds_libs_wh3n_U_g0t_reg3x?}
```