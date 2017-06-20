---
layout:     post
title:      "PicoCTF - Broadcast"
subtitle:   "Write-Ups"
date:       2017-04-18 0:00:00
author:     "W3ndige"
header-img: "img/picoctf-header.jpg"
permalink: /:title/
category: PicoCTF 2017
---
<h1>Broadcast - 120 PTS</h1>

<p>You stumbled upon a group Message. Can you figure out what they were sending? The string sent is ascii encoded as a hex number (submit the ascii string as the flag)</p>

{% highlight text %}
e = 3
c1 = 261345950255088824199206969589297492768083568554363001807292202086148198540785875067889853750126065910869378059825972054500409296763768604135988881188967875126819737816598484392562403375391722914907856816865871091726511596620751615512183772327351299941365151995536802718357319233050365556244882929796558270337
n1 = 1001191535967882284769094654562963158339094991366537360172618359025855097846977704928598237040115495676223744383629803332394884046043603063054821999994629411352862317941517957323746992871914047324555019615398720677218748535278252779545622933662625193622517947605928420931496443792865516592262228294965047903627
c2 = 147535246350781145803699087910221608128508531245679654307942476916759248311896958780799558399204686458919290159543753966699893006016413718139713809296129796521671806205375133127498854375392596658549807278970596547851946732056260825231169253750741639904613590541946015782167836188510987545893121474698400398826
n2 = 405864605704280029572517043538873770190562953923346989456102827133294619540434679181357855400199671537151039095796094162418263148474324455458511633891792967156338297585653540910958574924436510557629146762715107527852413979916669819333765187674010542434580990241759130158992365304284892615408513239024879592309
c3 = 633230627388596886579908367739501184580838393691617645602928172655297372145912724695988151441728614868603479196153916968285656992175356066846340327304330216410957123875304589208458268694616526607064173015876523386638026821701609498528415875970074497028482884675279736968611005756588082906398954547838170886958
n3 = 1204664380009414697639782865058772653140636684336678901863196025928054706723976869222235722439176825580211657044153004521482757717615318907205106770256270292154250168657084197056536811063984234635803887040926920542363612936352393496049379544437329226857538524494283148837536712608224655107228808472106636903723
{% endhighlight %}

<p>Another RSA challenge. This time we have number of group messages, encrypted with the same small exponent (e). After doing a little bit of googling, I've found that Hastad's Broadcast Attack is able to break this encryption. You can read more about it <a href="http://crypto.stanford.edu/~dabo/papers/RSA-survey.pdf">here</a>, at chapter 4.2. </p>

<p><i>If three parties participating in the same system encrypt the same message m using the same public exponent e=3, although perhaps different modulus n1, n2, and n3, then one can easily compute m from the three cipher texts.</i></p>

<p>Now we can look at the equations: </p>

<p><b>c1 = m^3 modulo n1</b></p>
<p><b>c2 = m^3 modulo n2</b></p>
<p><b>c3 = m^3 modulo n3</b></p>

<p>But what now? This amazing piece of code will perform the attack for us. Let's take a look at it. </p>

{% highlight python %}
import sys
import binascii
from Crypto.PublicKey import RSA
from base64 import b64decode

if (len(sys.argv)<7):
    print "\t\n\nArg error: python rsaHastad.py <n0 File> <n1 File> <n2 File> <c0 File> <c1 File> <c2 File> [--decimal/--hex/--b64] [-v/--verbose]\n\n"
    exit()

print "\n"

print "\t~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~"
print "\t        RSA Hastad Attack         "
print "\t         JulesDT -- 2016          "
print "\t         License GNU/GPL          "
print "\t~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~"


def chinese_remainder(n, a):
    sum = 0
    prod = reduce(lambda a, b: a*b, n)

    for n_i, a_i in zip(n, a):
        p = prod / n_i
        sum += a_i * mul_inv(p, n_i) * p
    return sum % prod


def mul_inv(a, b):
    b0 = b
    x0, x1 = 0, 1
    if b == 1: return 1
    while a > 1:
        q = a / b
        a, b = b, a%b
        x0, x1 = x1 - q * x0, x0
    if x1 < 0: x1 += b0
    return x1

def find_invpow(x,n):
    high = 1
    while high ** n < x:
        high *= 2
    low = high/2
    while low < high:
        mid = (low + high) // 2
        if low < mid and mid**n < x:
            low = mid
        elif high > mid and mid**n > x:
            high = mid
        else:
            return mid
    return mid + 1

def parseC(argv, index, verbose):
    import string
    file = open(argv[index],'r')
    cmd = ' '.join(argv)
    fileInput = ''.join(file.readlines()).strip()
    if '--decimal' in cmd:
        if verbose:
            print "##"
            print "##",fileInput
            print "## Considered as decimal input"
            print "##"
        return long(fileInput)
    elif '--hex' in cmd:
        if verbose:
            print "##"
            print "##",fileInput
            print "## Considered as hexadecimal input"
            print "##"
        return long(fileInput,16)
    elif '--b64' in cmd:
        if verbose:
            print "##"
            print "##",fileInput
            print "## Considered as base64 input"
            print "##"
        return long(binascii.hexlify(binascii.a2b_base64(fileInput)),16)
    else:
        try:
            fileInput = long(fileInput)
            if verbose:
                print "##"
                print "##",fileInput
                print "## Guessed as decimal input"
                print "##"
            return long(fileInput)
        except ValueError:
            if all(c in string.hexdigits for c in fileInput):
                if verbose:
                    print "##"
                    print "##",fileInput
                    print "## Guessed as hexadecimal input"
                    print "##"
                return long(fileInput,16)
            else:
                if verbose:
                    print "##"
                    print "##",fileInput
                    print "## Guessed as base64 input"
                    print "##"
                return long(binascii.hexlify(binascii.a2b_base64(fileInput)),16)
            pass

def parseN(argv,index):
    file = open(argv[index],'r')
    fileInput = ''.join(file.readlines()).strip()
    try:
        fileInput = long(fileInput)
        return fileInput
    except ValueError:
        from Crypto.PublicKey import RSA
        return long(RSA.importKey(fileInput).__getattr__('n'))
        pass


if __name__ == '__main__':
    e = 3
    cmd = ' '.join(sys.argv)
    if '-v' in cmd or '--verbose' in cmd:
        verbose = True
    else:
        verbose = False
    n0 = parseN(sys.argv,1)
    n1 = parseN(sys.argv,2)
    n2 = parseN(sys.argv,3)

    c0 = parseC(sys.argv,4,verbose)
    c1 = parseC(sys.argv,5,verbose)
    c2 = parseC(sys.argv,6,verbose)
    n = [n0,n1,n2]
    a = [c0,c1,c2]

    result = (chinese_remainder(n, a))
    resultHex = str(hex(find_invpow(result,3)))[2:-1]
    print ""
    print "~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~"
    print "Decoded Hex :\n",resultHex
    print "---------------------------"
    print "As Ascii :\n",resultHex.decode('hex')
print "~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~"
{% endhighlight %}

<p>Shoutout to <a href="https://github.com/JulesDT">JulesDT</a>!</p>  

<p>And now it's time to finally crack this message. Firstly let's create files that will hold our ciphertext and modulus n values. </p>

{% highlight bash %}
w3ndige@W3ndige ~> echo "261345950255088824199206969589297492768083568554363001807292202086148198540785875067889853750126065910869378059825972054500409296763768604135988881188967875126819737816598484392562403375391722914907856816865871091726511596620751615512183772327351299941365151995536802718357319233050365556244882929796558270337" >> c0
w3ndige@W3ndige ~> echo "1001191535967882284769094654562963158339094991366537360172618359025855097846977704928598237040115495676223744383629803332394884046043603063054821999994629411352862317941517957323746992871914047324555019615398720677218748535278252779545622933662625193622517947605928420931496443792865516592262228294965047903627" >> n0
w3ndige@W3ndige ~> echo "147535246350781145803699087910221608128508531245679654307942476916759248311896958780799558399204686458919290159543753966699893006016413718139713809296129796521671806205375133127498854375392596658549807278970596547851946732056260825231169253750741639904613590541946015782167836188510987545893121474698400398826" >> c1
w3ndige@W3ndige ~> echo "405864605704280029572517043538873770190562953923346989456102827133294619540434679181357855400199671537151039095796094162418263148474324455458511633891792967156338297585653540910958574924436510557629146762715107527852413979916669819333765187674010542434580990241759130158992365304284892615408513239024879592309" >> n1
w3ndige@W3ndige ~> echo "633230627388596886579908367739501184580838393691617645602928172655297372145912724695988151441728614868603479196153916968285656992175356066846340327304330216410957123875304589208458268694616526607064173015876523386638026821701609498528415875970074497028482884675279736968611005756588082906398954547838170886958" >> c2
w3ndige@W3ndige ~> echo "1204664380009414697639782865058772653140636684336678901863196025928054706723976869222235722439176825580211657044153004521482757717615318907205106770256270292154250168657084197056536811063984234635803887040926920542363612936352393496049379544437329226857538524494283148837536712608224655107228808472106636903723" >> n2
{% endhighlight %}

<p>Now we can use provided code to break the crypto, and get the message. </p>

{% highlight bash %}
w3ndige@W3ndige ~> python2 hastad.py n0 n1 n2 c0 c1 c2  


	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	        RSA Hastad Attack         
	         JulesDT -- 2016          
	         License GNU/GPL          
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Decoded Hex :
62726f6164636173745f776974685f736d616c6c5f655f69735f6b696c6c65725f3430333332333030313931
---------------------------
As Ascii :
broadcast_with_small_e_is_killer_40332300191
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{% endhighlight %}

<p>Amazing crypto challenge, and we have another flag!</p>
