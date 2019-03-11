---
layout:     post
title:      "AngstromCTF - slots"
date:       2018-03-23 2:00:00
author:     "W3ndige"
permalink: /:title/
category: "AngstromCTF 2018"
---

In this challenge I'm going to show how focusing on only one attack vector may make you fail hard.

- Category: Misc
- Points: 90

{% highlight text %}
defund is building a casino empire. Break his slot machine, which is running at web.angstromctf.com:3002. Note: connect with netcat or an equivalent tool.
{% endhighlight %}

### Solution

At the beginning, we're given a source of an application running on certain port `web.angstromctf.com:3002`. By connecting to this port using `nc`, we're given a simple blackjack game, taking an user bet and then making the random choice.

{% highlight test %}
$ nc web.angstromctf.com 3002                                                           
Welcome to Fruit Slots!
We've given you $10.00 on the house.
Once you're a high roller, we'll give you a flag.
You have $10.00.
Enter your bet: 5
🍐 : 🍈 : 🍈
🍉 : 🍇 : 🍒 ◀
🍒 : 🍌 : 🍒
You lost everything.
Play more to become a high roller!
You have $5.00.
Enter your bet: 5
🍐 : 🍌 : 🍒
🍇 : 🍌 : 🍒 ◀
🍒 : 🍇 : 🍉
You lost everything.
You have no money left. Low roller.
{% endhighlight %}

But as we have the source code, let's take a look at it.

{% highlight bash %}
# coding: utf8

import random
import signal
import SocketServer

flag = open('flag.txt').read()

PORT = 3002

FRUITS = ['🍌', '🍒', '🍐', '🍈', '🍇', '🍊', '🍉']
COMBOS = {
	'🍌3': 1,
	'🍒2': 1,
	'🍐2': 3,
	'🍈2': 3,
	'🍇2': 3,
	'🍊2': 3,
	'🍉2': 3,
	'🍒3': 3,
	'🍐3': 10,
	'🍈3': 10,
	'🍇3': 10,
	'🍊3': 10,
	'🍉3': 10
}

def line():
	return [random.choice(FRUITS) for i in range(3)]

def payout(line):
	for fruit in line:
		combo = fruit + str(line.count(fruit)) #count the number of occuring fruits
		if combo in COMBOS:
			return COMBOS[combo]
	return 0

class incoming(SocketServer.BaseRequestHandler):
	def handle(self):
		req = self.request

		def receive():
			buf = ''
			while not buf.endswith('\n'):
				buf += req.recv(1)
			return buf[:-1]

		signal.alarm(60)

		req.sendall('Welcome to Fruit Slots!\n')
		req.sendall('Weve given you $10.00 on the house.\n')
		req.sendall('Once youre a high roller, well give you a flag.\n')

		money = 10
		while True:
			req.sendall('You have ${0:.2f}.\n'.format(money))
			req.sendall('Enter your bet: ')
			bet = receive()

			try:
				bet = float(bet)
			except:
				req.sendall('Your bet must be a number!\n')
				req.close()
				return

			if bet <= 0:
				req.sendall('Sneaky, but not good enough.\n')
				req.close()
				return
			elif bet > money:
				req.sendall('You dont have enough money to wager this.\n')
				req.close()
				return

			line1 = line()
			line2 = line()
			line3 = line()
			while payout(line2):
				line2 = line()
			win = bet * payout(line2)
			money += win - bet

			req.sendall('{}\n{} ◀\n{}\n'.format(' : '.join(line1), ' : '.join(line2), ' : '.join(line3)))

			if win > 0:
				req.sendall('You won ${0:.2f}!'.format(win))
			else:
				req.sendall('You lost everything.\n')

			if money <= 0:
				req.sendall('You have no money left. Low roller.\n')
				req.close()
				return
			elif money < 1000000000:
				req.sendall('Play more to become a high roller!\n')
			else:
				req.sendall('Wow, you\'re a high roller!\n')
				req.sendall('A flag: {}\n'.format(flag))
				return

class ReusableTCPServer(SocketServer.ForkingMixIn, SocketServer.TCPServer):
	pass

SocketServer.TCPServer.allow_reuse_address = True
server = ReusableTCPServer(('0.0.0.0', PORT), incoming)

print 'Server listening on port %d' % PORT
server.serve_forever()
{% endhighlight %}

From the start we can see that the input is converting our numbers to float with `bet = float(bet)`, then it performs checks that will determine whether the numbers isn't smaller than 0, and if it's not bigger than our current money status.

After that the application will generate three lines, with the second/middle one used to generate our outcome.

{% highlight python %}
line1 = line()
line2 = line() # return [random.choice(FRUITS) for i in range(3)]
line3 = line()
{% endhighlight %}

But as the random number generator isn't seeded with any value, the sequence of values should be the same for each run of this program. That's what I decided to dig into and check whether or not it's correct. Below are the sample three 3 games.

{% highlight text %}
$ nc web.angstromctf.com 3002                                                           
Welcome to Fruit Slots!
We've given you $10.00 on the house.
Once you're a high roller, we'll give you a flag.
You have $10.00.
Enter your bet: 3
🍐 : 🍈 : 🍈
🍉 : 🍇 : 🍒 ◀
🍒 : 🍌 : 🍒
You lost everything.
Play more to become a high roller!
You have $7.00.
Enter your bet: 3
🍐 : 🍌 : 🍒
🍇 : 🍌 : 🍒 ◀
🍒 : 🍇 : 🍉
You lost everything.
Play more to become a high roller!
You have $4.00.
Enter your bet: 4
🍉 : 🍇 : 🍇
🍌 : 🍐 : 🍉 ◀
🍊 : 🍈 : 🍈
You lost everything.
You have no money left. Low roller.

$ nc web.angstromctf.com 3002                                                           
Welcome to Fruit Slots!
We've given you $10.00 on the house.
Once you're a high roller, we'll give you a flag.
You have $10.00.
Enter your bet: 3
🍐 : 🍈 : 🍈
🍉 : 🍇 : 🍒 ◀
🍒 : 🍌 : 🍒
You lost everything.
Play more to become a high roller!
You have $7.00.
Enter your bet: 3
🍐 : 🍌 : 🍒
🍇 : 🍌 : 🍒 ◀
🍒 : 🍇 : 🍉
You lost everything.
Play more to become a high roller!
You have $4.00.
Enter your bet: 4
🍉 : 🍇 : 🍇
🍌 : 🍐 : 🍉 ◀
🍊 : 🍈 : 🍈
You lost everything.
You have no money left. Low roller.

$ nc web.angstromctf.com 3002                                                           
Welcome to Fruit Slots!
We've given you $10.00 on the house.
Once you're a high roller, we'll give you a flag.
You have $10.00.
Enter your bet: 3
🍐 : 🍈 : 🍈
🍉 : 🍇 : 🍒 ◀
🍒 : 🍌 : 🍒
You lost everything.
Play more to become a high roller!
You have $7.00.
Enter your bet: 3
🍐 : 🍌 : 🍒
🍇 : 🍌 : 🍒 ◀
🍒 : 🍇 : 🍉
You lost everything.
Play more to become a high roller!
You have $4.00.
Enter your bet: 4
🍉 : 🍇 : 🍇
🍌 : 🍐 : 🍉 ◀
🍊 : 🍈 : 🍈
You lost everything.
You have no money left. Low roller.
{% endhighlight %}

As you can see, values do not change with each game. That made me feel confident and that's how I stopped further analyzing code and decided to dig into this problem, writing python script that will bet very small amounts of money in order to check where it's possible to win some money. In quick words - ***bad idea***.

That's why:

{% highlight python %}
while payout(line2):
  line2 = line()
{% endhighlight %}

This loop will iterate as long as we're able to win something, making sure that it's impossible to win from any combo in this game. Banging my head after such a stupid mistake, I decided to look for something else in this code and in [Python documentation](https://docs.python.org/2/library/functions.html#float) about `float()` function I've noticed something interesting.

{% highlight text %}
Note

When passing in a string, values for NaN and Infinity may be returned, depending on the underlying C library. Float accepts the strings nan, inf and -inf for NaN and positive or negative infinity. The case and a leading + are ignored as well as a leading - is ignored for NaN. Float always represents NaN and infinity as nan, inf or -inf.
{% endhighlight %}

Interesting, let's check the `NaN` value, as the `-+inf` will be blocked by the boundary checks.

{% highlight text %}
$ nc web.angstromctf.com 3002                                                           
Welcome to Fruit Slots!
We've given you $10.00 on the house.
Once you're a high roller, we'll give you a flag.
You have $10.00.
Enter your bet: NaN
🍐 : 🍈 : 🍈
🍉 : 🍇 : 🍒 ◀
🍒 : 🍌 : 🍒
You lost everything.
Wow, you're a high roller!
A flag: actf{fruity}
{% endhighlight %}
