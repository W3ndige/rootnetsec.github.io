---
layout:     post
title:      "Problem 37 - Truncatable Primes"
subtitle:   "Project Euler Solutions"
date:       2016-10-15 2:00:00
author:     "W3ndige"
header-img: "img/euler-header.jpg"
category: Programming
---

<h1>Introduction</h1>

<p>Today I'm gonna try and solve another challenge from <a href="https://projecteuler.net">Project Euler</a> - site with various programming tasks heavily connected with mathematics. If you're learning any new programming language and don't have any ideas for new programs - just head over to this site, complete as many challenges as you can and you'll definitely improve your skills. Just take a look at this task: </p>

{% highlight text %}
The number 3797 has an interesting property. Being prime itself, it is possible to continuously remove digits from left to right, and remain prime at each stage: 3797, 797, 97, and 7. Similarly we can work from right to left: 3797, 379, 37, and 3.

Find the sum of the only eleven primes that are both truncatable from left to right and right to left.

NOTE: 2, 3, 5, and 7 are not considered to be truncatable primes
{% endhighlight %}

<h1>How to start?</h1>

<p>Best way to solve any problem is to divide it into many smaller ones - that way you will have to 'glue' these parts together into one and you'll hopefully have fully functioning solution. In this problem I want to firstly complete this smaller problems:  </p>

<ol>
<li>Check if a number is prime</li>
<li>Truncate the number from left and from right</li>
<li>Check if every result from truncating is also a prime</li>

</ol>

<p><b>As I'm currently learning C++ this solution will be written in this language</b>, but you can easily convert it to the Python version after understanding the concept. Completing these 3 problems will let us complete this challenge so without wasting time let's move to the first one. </p>

<h1>Prime check</h1>

<p>How to check if a number is a prime? Consider this example: </p>

<p>We've got some number <b>x</b> that we want to check. Now if we run a loop from 1 to the x, then on every iteration perform operation <b>x modulo i</b>. If the result is different than 0 for every number, then it's a prime, if not then it's not. </p>

<p>But as you may think, it will be really slow for bigger numbers, so we will have to speed up this algorithm. Firstly check if the number is smaller or equall to one, if so this number cannot be prime (remember that 1 is not a prime). Then assure that 2 and 3 are prime numbers. Also if you take a look at group of prime numbers, none of them are divisable by 2 so quickly checking it in a first place will make it run a little bit faster. Lastly run the checking loop. </p>

{% highlight cpp %}
bool isPrime(int x) {
  //Check if a number is prime

  if(x <= 1) return false;
  if(x == 2 || x == 3) return true;
  else if (x % 2 == 0) return false;

  else {

      for (int i = 3; i < x; i++) {

          if (x % i == 0) return false;
      }
  }

  return true;
}
{% endhighlight %}

<b>Quick note from the future!</b>

<p>This version of algorithm took a few minuts to finish, but without any ideas how to make it faster, I looked at some other algorithms. That's the improved version of the checking loop: </p>

{% highlight cpp %}

for (int i = 3; i < (int)sqrt((float)x)+1; i++) {

    if (x % i == 0) return false;
}

{% endhighlight %}
<p><i>Runs in around second ;)</i></p>

<h1>Truncatable Check</h1>

<p>Now we have to check, if after removing every digit from left, or right, it will still be a prime. If that was Python I would write this function in a few lines but not today, not that easy ;)</p>

<p>After tinkering with it for a little bit I came up with this idea. If you have integer ex 152 and divide it by 10 then the result will be 15. Same as removing the last number, right? Remember that you have to do this on integers as if it was floating point you would result in having 15.2. Then 15 / 10 = 1. So we've got idea for removing from the right, now let's think about the left side.  </p>

<p>And here comes modulo - to rescue us! Let's have 152 as example once again. 152 modulo 100 = 52, equivalent to removing the first number. Then 152 % 10 = 2 and operation is completed. It's very helpful to remember these tricks as they may come handy in the most unexpected moments. </p>

{% highlight cpp %}
bool truncatableCheck(int x) {
    if (x<=0) return false;

    //Get the length of the number

    int len = 0;
    int tmp = x;
    while (tmp !=0)
    {
        len ++;
        tmp /= 10;
    }

    //Truncate from right and check if is prime

    tmp = x;
    while (tmp != 0) {

        if (isPrime(tmp) != true) {

            return false;
        }

        tmp /= 10;
    }

    //Truncate from left and check if is prime

    tmp = x;
    while(len > 1 ) {

        int y = tmp % (int)pow((float)10, len-1);

        if (isPrime(y) != true) {
            return false;
        }

        len--;
    }

    return true;
}
{% endhighlight %}

<h1>Main</h1>

<p>See how we finished all smaller problems? Now it's time to connect them all together in the main function. </p>

{% highlight cpp %}
int main() {

      int numbers = 0;
      int sum = 0;
      int i = 8;

      while(numbers < 11) {

          if (truncatableCheck(i)) {
              sum += i;
              numbers ++;
              cout << i << endl;
          }
          i++;
      }
      cout << "Sum "<< sum << endl;
      return 0;
}
{% endhighlight %}

<p>As we know that there's only eleven primes we can only iterate the loop eleven times. If the number is truncatable we will print it's value and we'll add it to the sum of all truncatable primes. </p>

<p>Remember about essential libraries - <b>cmath</b> and <b>iostream</b>, and we're ready to go. </p>

{% highlight bash %}
$ ./truncatable_primes23
37
53
73
313
317
373
797
3137
3797
739397
Sum 748317
{% endhighlight %}

<p>Great - it works and in addition it works quite fast so I'm happy with this result. As you probably know you can try to approach this problem in different ways so don't worry if you came up with something different and/or better. For more project euler challenges check out my GitHub <b><a href="https://github.com/W3ndige/coding-challenges">W3ndige</a></b> </p>

<hr>
<p>As a sidenote I would like to apologize for the smaller amount of content - I lack free time as I'm concentrating on programming and mathematics to pass my exams, but don't worry - I'm working on a few netsec connected posts and projects so keep tuned for more!  </p>

<p>~ Stay safe! </p>
