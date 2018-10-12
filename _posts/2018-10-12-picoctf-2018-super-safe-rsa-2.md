---
layout:     post
title:      "PicoCTF 2018 - Super Safe RSA 2"
date:       2018-10-12 1:00:00
author:     "W3ndige"
permalink: /:title/
category: PicoCTF 2018
---

{% highlight text %}
 Wow, he made the exponent really large so the encryption MUST be safe, right?! Connect with nc 2018shell2.picoctf.com 29483. 
{% endhighlight %}

### Solution

```
[w3ndige@main ~]$ nc 2018shell2.picoctf.com 29483
c: 38100535198911824685119778176240613907792115289783297714119085321825251980567271097679045239915771759956639435046839702504012549411196449182671875480634892536495979225290685939648356967395298770295308702729226838548468583083497350478075425293805710712861517891524352434308368518530989727752982070216077584640
n: 87280956711935692317474727224374956470336036497971493795576316783854895896025695054090305813573095080821954868224476553410323819494183302445148576100546354012567295329260075496985357430102529913793811212942095608098482238937596993960766684593691418084218467418822948271740665585515173755486913423931373736467
e: 48358313916827088877120173646097304490507222199335381711264959316699805070122053391352565640731993461399301207258479456351103471468838822269646000683384011718128424291475290697395623321388678873440420222001418182317143522503857299697279421917746266748420824069937203914235656756792123573836983617702339928193
```

Once again in the PicoCTF cometition we are presented with a RSA challenge with a very high exponent. We can suppose that this is ***Wiener's attack***, as stated in this [paper](https://www.iacr.org/archive/pkc2004/29470001/29470001.pdf). 

>In the RSA Cryptosystem, Bob might tend to use a small value of d, rather than a large random number to improve the RSA decryption performance. However, Wiener’s attack shows that choosing a small value for d will result in an insecure system in which an attacker can recover all secret information, i.e., break the RSA system. 

We attack this type of challenge in the previous edition PicoCTF 2017, so if you want to technique will be the same as in [this post](https://www.rootnetsec.com/picoctf-smallrsa/). 

Firstly, let's clone the the [repo](https://github.com/pablocelayes/rsa-wiener-attack) made by ***pablocelayes***. There we can modify script called `RSAwienerHacker.py` in order to attack our variables, not the ones that the script generated. 

```python
[w3ndige@main rsa-wiener-attack]$ cat RSAwienerHacker.py 
'''
Created on Dec 14, 2011

@author: pablocelayes
'''

import ContinuedFractions, Arithmetic, RSAvulnerableKeyGenerator

def hack_RSA(e,n):
    '''
    Finds d knowing (e,n)
    applying the Wiener continued fraction attack
    '''
    frac = ContinuedFractions.rational_to_contfrac(e, n)
    convergents = ContinuedFractions.convergents_from_contfrac(frac)
    
    for (k,d) in convergents:
        
        #check if d is actually the key
        if k!=0 and (e*d-1)%k == 0:
            phi = (e*d-1)//k
            s = n - phi + 1
            # check if the equation x^2 - s*x + n = 0
            # has integer roots
            discr = s*s - 4*n
            if(discr>=0):
                t = Arithmetic.is_perfect_square(discr)
                if t!=-1 and (s+t)%2==0:
                    print("Hacked!")
                    return d

# TEST functions

def test_hack_RSA():
    print("Testing Wiener Attack")
    times = 5
    
    while(times>0):
        #e,n,d = RSAvulnerableKeyGenerator.generateKeys(1024)
        e = 48358313916827088877120173646097304490507222199335381711264959316699805070122053391352565640731993461399301207258479456351103471468838822269646000683384011718128424291475290697395623321388678873440420222001418182317143522503857299697279421917746266748420824069937203914235656756792123573836983617702339928193
        n = 87280956711935692317474727224374956470336036497971493795576316783854895896025695054090305813573095080821954868224476553410323819494183302445148576100546354012567295329260075496985357430102529913793811212942095608098482238937596993960766684593691418084218467418822948271740665585515173755486913423931373736467
        hacked_d = hack_RSA(e, n)
    
        print("hacked_d = ", hacked_d)
        print("-------------------------")
        times -= 1
    
if __name__ == "__main__":
    #test_is_perfect_square()
    #print("-------------------------")
    test_hack_RSA()
```

Right after that we can run it. 

```
[w3ndige@main rsa-wiener-attack]$ python RSAwienerHacker.py 
Testing Wiener Attack
Hacked!
hacked_d =  65537
-------------------------
Hacked!
hacked_d =  65537
-------------------------
Hacked!
hacked_d =  65537
-------------------------
Hacked!
hacked_d =  65537
-------------------------
Hacked!
hacked_d =  65537
-------------------------
``` 

Wow, seems that author of the challenge most likely exchanged the values, as 65537 is usually used for exponent. 

But with that knowledge we can decrypt the ciphertext. 

```python
>>> d = 65537
>>> n = 87280956711935692317474727224374956470336036497971493795576316783854895896025695054090305813573095080821954868224476553410323819494183302445148576100546354012567295329260075496985357430102529913793811212942095608098482238937596993960766684593691418084218467418822948271740665585515173755486913423931373736467
>>> c = 38100535198911824685119778176240613907792115289783297714119085321825251980567271097679045239915771759956639435046839702504012549411196449182671875480634892536495979225290685939648356967395298770295308702729226838548468583083497350478075425293805710712861517891524352434308368518530989727752982070216077584640
>>> plain = str(hex(pow(c, d, n)))[2::]
>>> print(''.join([chr(int(''.join(c), 16)) for c in zip(plain[0::2],plain[1::2])]))
picoCTF{w@tch_y0ur_Xp0n3nt$_c@r3fu11y_5495627}
```

And another flag is ours!