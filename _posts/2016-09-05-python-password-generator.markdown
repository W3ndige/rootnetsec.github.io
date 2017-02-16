---
layout:     post
title:      "Simple Password Generator"
subtitle:   "Using Python Programming Language"
date:       2016-09-05 12:00:00
author:     "W3ndige"
header-img: "img/python-password-generator-header.png"
---

<h1>Introduction</h1>
<p>Hi everyone!</p>
<p>As a part of my weekly coding challenges I decided to create simple password generating program that allows an user to create strong and unique passwords depending on the wanted legth. It's great way to improve your programming skills and problem solving. Let's see how it works! </p>

<h1>Code</h1>

<p>I will break down the code of the program into smaller pieces that will let you step by step understand how it works. We're going to use 2 built in libraries - <b>random</b> to create as random passwords as it's possible and <b>string</b> to let us work on the characters used in password.</p>

{% highlight python %}
import random
import string
{% endhighlight %}

<b>Main Function</b>

<p>Firstly, the main function should ask the user for the length of the password he wants to create - no one will use a program that will only generate passwords made of 10 characters. But we should also remember that the user input should be checked whether or not  it's a digit since we don't want the password of length "t".  </p>

{% highlight python %}
def main():
    print("[*] Welcome to Password Generator")
    print("[*] Remember that strong password should be longer than 8 characters!")
    length = input("[*] Enter how long would you like your password to be: ")
    if length.isdigit():
        length = int(length)
    else:
        print("[*] Please enter a number!")
        return

{% endhighlight %}

<p>But what next? As our program generates only strong passwords made of lowercase and uppercase letters, symbols and numbers and then checks for at least one of each we should consider the minimum length of the password. Any password shorter or equal to 3 character will never be generated, and we don't want to make the user sit in front of the screen for hours and hours just to generate short password. </p>

{% highlight python %}
if length <= 3:
    print("[*] It's not possible to generate the password. Please use longer one. ")
else:
    password_strong = False
    while not password_strong:
        password = generate(length)
        password_strong = check(password)
    print("[*] Your password has been generated: " + ''.join(password))
{% endhighlight %}

<p>Okay, so now we have the filter that won't allow password shorter or equal to 3 characters but what happens after it? Firstly we define that password_strong is false as we haven't checked it yet. But what happens next? </p>
<ul>
<li><b>While not False:</b> results in <b>while True:</b></li>
<li>Then the loop proceeds to execute the following block of code: generating the password and checking for it's complexity </li>
<li>If the check ends in <b>False</b> then the loop goes on, meaning that the password has been generated but it was missing essential elements to pass the check. </li>
<li>If the check ends in <b>True</b> then the loop ends, meaning that the password has been generated and it's the strong one.</li>
</ul>

<p>After generating the strong password, program prints it on the screen. </p>

<b>Generate Function</b>

<p>Now let's explore the most essential function in this program which generates a password. </p>

{% highlight python %}
def generate(length):
    password = ''.join(random.SystemRandom().choice(string.ascii_letters + string.punctuation + string.digits) for _ in range(length))
    return password
{% endhighlight %}

<p>Let's see how it works step by step. Firstly we have an empty string in which we're going to add our password. After that we're going to use random choice which basically will create a random character from the predefined set of characters (like string.ascii_letters which are all letters uppercase and lowercase)  "length" number of times. So if we have length 4 it will four times get the random character from the random set and then it will add it to the string. Use of <b>random.SystemRandom</b> will make sure that these are cryptographically secure PRNGs. Easy, right? </p>

<b>Check Function</b>

<p>This function is pretty self explanatory, it checks whether or not password is made from at least one uppercase letter, one lowercase letter, one digit and one special character. It returns True if the password is secure and False if it's not.  </p>

{% highlight python %}
def check(password):
    lower = sum(1 for letter in password if letter.islower())
    upper = sum(1 for letter in password if letter.isupper())
    digit = sum(1 for letter in password if letter.isnumeric())
    special = sum(1 for letter in password if letter in string.punctuation)

    if lower > 0 and upper > 0 and digit > 0 and special > 0:
        return True
    else:
        return False
{% endhighlight %}

<p>That's why I love python, even without knowing programming any person could interpret the code. It sums the number of each parameter by checking if it's in the certain set, letter by letter. If it occurs, it will add to the sum 1, if not the sum will stay the same. Then it checks if the password contains at least one of each character type, resulting in <b>True</b> if the password is strong and <b>False</b> if it's not strong one.  </p>

<p>At the end I added the: </p>

{% highlight python %}
if __name__ == '__main__':
    main()
{% endhighlight %}

<p>By doing the main check, you can have that code only execute when you want to run the module as a program and not have it execute when someone just wants to import your module and call your functions themselves. Check out whole code at my Github account. </p>

[Github](https://github.com/W3ndige/coding-challenges/blob/master/password_generator.py "Github")<br>

<p>I hope you enjoyed exploring the concept behind this small program. Remember that doing a lot of small challenges like that one will help you a lot in learning programming. Good luck with creating your own versions! </p>

<p>~ Stay safe!</p>
