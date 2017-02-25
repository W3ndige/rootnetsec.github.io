---
layout:     post
title:      "Three Coding Challenges"
subtitle:   "Largest Prime Factor, Largest Palindrom Product and 10001st Prime"
date:       2016-09-10 12:00:00
author:     "W3ndige"
header-img: "img/coding-challenges-1-header.jpg"
category: Programming
---

<h1>Introduction</h1>

<p>Hi, today we're gonna explore more about programming and math. I've chosen 3 challenges from <a href="https://projecteuler.net/">Project Euler</a>- Largest Prime Factor of a number, Largest Palindrom and 10001st Prime. Project Euler is a great site that provides us with many mathematical/programming problems that will undoubtedly help us gain a better insight in beautiful word of mathematics and programming. </p>

<p>I strongly encourage you to come up with your own ideas as you will learn more than just by reading about this. I have spent quite a few hours working on the solutions and I can say without hesitation, that the feeling  after finishing the task is amazing. But let's move on to the tasks! ;)</p>

<h1>Largest Prime Factor</h1>
<p>In this task we are assigned to find the largest prime factor of a number 600851475143. Quite a big one, right? </p>
<p>Firstly let's see how finding prime factors works on some smaller number, let's say 12. </p>
<ol>
<li>Divide the number by the smallest prime number which is 2 |  <b>12 / 2 = 6</b> | <b>Prime factor = 2</b></li>
<li>Divide the the previous result by 2 | <b>6 / 2 = 3</b> | <b>Prime factor = 2 and Prime factor = 3</b></li>
<li>Prime factors of number 12 are 2 , 2, 3 because <b>12 = 2 * 2 * 3</b></li>
</ol>

<p>If you still don't know how it works let's see another example - number 147</p>
<ol>
<li>In this example dividing by 2 will not work beacuse 147 / 2 = 73,5 so let's change divider to the next prime number which is 3 | <b>147 / 3 = 49</b> | <b>Prime factor = 3</b></li>
<li>Now let's try factoring 49. Smallest number that will work is 7. | <b>49 / 7 = 7</b> | <b>Prime factor = 7 and Prime factor = 7</b></li>
<li>Prime factors of number 147 are 3 , 7, 7 because <b>147 = 3 * 7 * 7</b></li>
</ol>

<p>But how to implement it with python code? That's my solution. </p>
{% highlight python %}
def prime_factors(x):
    a = 2
    factors = []
    while a * a <= x:
        if x % a:
            a += 1
        else:
            x //= a
            factors.append(a)
    if x > 1:
        factors.append(x)
    return factors

factors = prime_factors(600851475143)

print(factors)
print(max(factors))

{% endhighlight %}

<p>Firstly we define the variable <b>a</b> which is starting value of our divider and list <b>factors</b> that will hold our prime factors.  Then inside of the while loop we are checking if our desired number is divisible by <b>a</b>, if not we're adding 1 to <b>a</b>, if yes we're adding <b>a</b> to the list of factors and we're changing number <b>x into the x // a</b>. At the end of the loop, program will return list of factors, and then print the biggest one using <b>max</b> function.  </p>

{% highlight python %}
$ python3 prime_factors.py
[71, 839, 1471, 6857]
6857
{% endhighlight %}

<h1>Largest Palindrom Product</h1>

<p>Palindrom is a number that is read the same from the beginning as from the end, for example 2506052. In this task our goal is to find the largest palindrome made from the product of two 3-digit numbers. For me the hardest thing is to check if the number is the same from it's beginning as from it's end because generating such a number will be just brute forcing every combination. </p>

{% highlight python %}
def find_palindrome():
    largest = 0
    for i in range (100, 999):
        for r in range(100, 999):
            x = str(i * r)
            y = x[::-1]
            if x == y:
                x = int(x)
                if x > largest:
                    largest = (x)
    return largest

def main():
    print(find_palindrome())

if __name__ == '__main__':
    main()
{% endhighlight %}

<p>Firstly we're going to create a range for both of the numbers <b>i</b> and <b>r</b> from 100 to 999 (minimal and maximal 3 digit number). Then <b>x</b> is going to be the result of multiplying both of these numbers <b>i</b> and <b>r</b> and it's going to be our palindrome - yet we need to check if it's really palindrome. But how to check it? I used <b>x[::-1]</b> which will generate string in the reverse order. Here's how it works:</p>
{% highlight python %}
>>> string = "1234"
>>> print(string[::-1])
4321
{% endhighlight %}

<p>Then if the number is equal to its reverse version we're changing it into the integer and assigning it to the variable <b>largest</b>. Last part of the code will check whether or not the palindrome is the largest, if not, it's going to generate the bigger one, if yes, then it's getting printed. </p>
{% highlight python %}
$ python3 palindrome.py
906609
{% endhighlight %}

<h1>10001st Prime</h1>

<p>And here comes the last one. We have to find the 10001st prime number. By listing the first six prime numbers: 2, 3, 5, 7, 11, and 13, we can see that the 6th prime is 13. After thinking for a while I came up with this simple solution to the problem using previous <b>prime_factors</b> function. Prime number is only divisible by itself and 1. So there's going to be only one factor of this number - itself. That's how I want to check whether or not it's a prime number. Let's take a look at the code. </p>

{% highlight python %}
def prime_factors(x):
    a = 2
    factors = []
    while a * a <= x:
        if x % a:
            a += 1
        else:
            x //= a
            factors.append(a)
    if x > 1:
        factors.append(x)
    return factors


def generate():
    exists = False
    number = 0
    counter = 0
    while not exists:
        number += 1
        factors = prime_factors(number)
        if len(factors) == 1 and factors[0] == number:
            counter += 1
            if counter == 10001:
                print(number)
                exists = True
        else:
            pass
if __name__ == '__main__':
    generate()
{% endhighlight %}

<p>We're using our previous function to generate prime factors of a number. In the <b>generate()</b> function loop is going to iterate numbers from 1 until we find desired prime number, then the loop is going to stop. Counter will allow us to see which prime number it is. Then if there is only one factor and it's value is the same as the number counter is incrementing by one. After that, when the counter reaches 10001st number, program will print the result and loop will end. Quite simple, right? </p>

{% highlight python %}
$ python3 prime_10001.py
104743
{% endhighlight %}

[Github](https://github.com/W3ndige/coding-challenges "Github")<br>

<p>And that's the last one. I strongly encourage you to try other tasks as I'm sure that it will significantly improve your programming and mathematical knowledge. As always thanks for reading, it really motivates me to write more about. Good luck! </p>

<p>~ Stay safe!</p>
