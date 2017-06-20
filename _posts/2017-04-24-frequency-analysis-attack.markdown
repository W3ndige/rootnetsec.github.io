---
layout:     post
title:      "Frequency Analysis"
subtitle:   "Crypto Attack"
date:       2017-04-24 0:00:00
author:     "W3ndige"
header-img: "img/frequency-analysis-header.jpeg"
permalink: /:title/
category: Cryptography
---

<h1>Introduction</h1>

<p><b>Frequency analysis</b> is the study of the frequency of letters or groups of letters occuring next to each other. The most ancient description for what we know was made by Al-Kindi, dating back to the IXth century. This attack is used to break <b>monoalphabetic ciphers</b>, which work by simple and fixed substitution of letters. We now, that if the <b>a</b> in plaintext is encrypted to <b>x</b> letter, everytime in the ciphertext that <b>x</b> will convert to <b>a</b>.  </p>

<p>But how can we attack the ciphertext without knowing the encryption key? Brute force? That may take too long for some, even basic, forms of encryption. What comes to the rescue is frequency analysis. With knowledge of how often each letter occurs in the language, we can try analyzing the ciphertext, substituting most occuring ones with the ones that are most frequent in specified language. Let's take a look at charts showing those frequencies in English. </p>

![Letter frequency in English](/img/frequency-analysis/english.png){:class="img-responsive center-block"}

<p>As we now can see, it's easy to determine which letter is the most common, which least. We have to remember that for the correct analysis we'll need as much text as possible, as it will give more accurate results. In addition we know that there are many comman letter pairs:  </p>

<p><b>TH, EA, OF, TO, IN, IT, IS, BE, AS, AT, SO, WE, HE, BY, OR, ON, DO, IF, ME, MY, UP </b></p>

<p>Repeated letters: </p>

<p><b>SS, EE, TT, FF, LL, MM and OO </b></p>

<p>Common triplets: </p>

<p><b>THE, EST, FOR, AND, HIS, ENT or THA </b></p>

<p>Now let's use this knowledge to show how this attack works. </p>

<h1>Attack</h1>

<p>Let's consider this message: </p>
<p><i>KRBGXZ QHEG ZGLXQT PRZUJHXGU LWLHZUV VNG PRQQGPVHFG. OG LXG VRXKGZVGB DT L XGQGZVQGUU EQRO RE HZERXKLVHRZ LU OGQQ LU VNG BLHQT ORXXHGU RE LZ GVGXZLQQT HZUGPYXG, YZOLXXLZVGB QHEG. EYXVNGXKRXG, OG BXGLB VNG VNRYWNV RE DGHZW LQHMG, RE UNLXHZW KYQVHJQG FHGOU LZB RJHZHRZU. LU UYPN, OG LXG VYXZHZW JXRWXGUUHFGQT CYBWGKGZVLQ RE ONR OG UNRYQB DG JLXVZGXHZW OHVN, RZ VNG DLUHU VNLV "VNGT BR ZRV YZBGXUVLZB". HZ NLPMHZW, HV TGV HKJQHPLVGU RZ VNG BGQHPLVG UYDCGPV RE VXYUV, ONHPN ORYQB XGIYHXG LZ GUULT RZ HVUGQE, WHFGZ VNG YZBGZHLDQG HKJRXVLZPG VNG KLVVGX NLU LPIYHXGB RFGX VNG TGLXU.</i></p>

<p>Firstly let's analyze the frequency of letters. I've used this <a href="http://md5decrypt.net/en/Letters-frequency-analysis/">website</a> to create a graph.</p>

![Letter frequency in message](/img/frequency-analysis/message-frequency.png){:class="img-responsive center-block"}

<p>As we now know that <b>G</b> is the most common letter, let's change it to <b>E</b>. Then we can also change <b>V</b> to <b>T</b> as it's second most common letter. After this operation we can definitely see more clues. </p>

<p><i>tNe</i> word is definitely word <b>the</b>, so let's change <b>N</b> to <b>H</b>. Now we can also change the <i>thLt</i> to <b>that</b>, meaning <b>L</b> -> <b>A</b> and word <i>theT</i> to <b>then</b>, so <b>T</b> to <b>N</b>. Remember to keep these replacings case sensitive!  </p>

<p><i>KRBeXZ QHEe ZeaXQn PRZUJHXeU aWaHZUt the PRQQePtHFe. Oe aXe tRXKeZteB Dn a XeQeZtQeUU EQRO RE HZERXKatHRZ aU OeQQ aU the BaHQn ORXXHeU RE aZ eteXZaQQn HZUePYXe, YZOaXXaZteB QHEe. EYXtheXKRXe, Oe BXeaB the thRYWht RE DeHZW aQHMe, RE UhaXHZW KYQtHJQe FHeOU aZB RJHZHRZU. aU UYPh, Oe aXe tYXZHZW JXRWXeUUHFeQn CYBWeKeZtaQ RE OhR Oe UhRYQB De JaXtZeXHZW OHth, RZ the DaUHU that "then BR ZRt YZBeXUtaZB". HZ haPMHZW, Ht net HKJQHPateU RZ the BeQHPate UYDCePt RE tXYUt, OhHPh ORYQB XeIYHXe aZ eUUan RZ HtUeQE, WHFeZ the YZBeZHaDQe HKJRXtaZPe the KatteX haU aPIYHXeB RFeX the neaXU.</i></p>

<p>Here's our message as in current state. Do we have anything more? Word <i>haU</i> can be possibly <b>has</b> so we have another <b>U</b> to <b>S</b>. Then, we have <i>Dn</i>, where <b>D</b> would be <b>i</b>. Verb <b>aXe</b>, will change <b>X</b> into <b>R</b>. </p>

<p>After that operation, word <i>trYst</i> appears, meaning that <b>Y</b> can be <b>U</b>. Now maybe <i>Oe</i> can be a word <b>we</b>? </p>

<p><i>KRBerZ QHEe ZearQn PRZsJHres aWaHZst the PRQQePtHFe. we are tRrKeZteB in a reQeZtQess EQRw RE HZERrKatHRZ as weQQ as the BaHQn wRrrHes RE aZ eterZaQQn HZsePure, uZwarraZteB QHEe. EurtherKRre, we BreaB the thRuWht RE ieHZW aQHMe, RE sharHZW KuQtHJQe FHews aZB RJHZHRZs. as suPh, we are turZHZW JrRWressHFeQn CuBWeKeZtaQ RE whR we shRuQB ie JartZerHZW wHth, RZ the iasHs that "then BR ZRt uZBerstaZB". HZ haPMHZW, Ht net HKJQHPates RZ the BeQHPate suiCePt RE trust, whHPh wRuQB reIuHre aZ essan RZ HtseQE, WHFeZ the uZBeZHaiQe HKJRrtaZPe the Katter has aPIuHreB RFer the nears.</i></p>

<p>Can you see the word <i>turZHZW</i>? I think it can be a word <b>turning</b>, as the letters would be correct, but that means that we have issue somewhere in the previous operations. I try to exchange this letters with uppercase, so we would now that it's our guess. </p>

<p><i>KRBerN QIEe NearQn PRNsJIres aGaINst the PRQQePtIFe. we are tRrKeNteB in a reQeNtQess EQRw RE INERrKatIRN as weQQ as the BaIQn wRrrIes RE aN eterNaQQn INsePure, uNwarraNteB QIEe. EurtherKRre, we BreaB the thRuGht RE ieING aQIMe, RE sharING KuQtIJQe FIews aNB RJINIRNs. as suPh, we are turNING JrRGressIFeQn CuBGeKeNtaQ RE whR we shRuQB ie JartNerING wIth, RN the iasIs that "then BR NRt uNBerstaNB". IN haPMING, It net IKJQIPates RN the BeQIPate suiCePt RE trust, whIPh wRuQB reIuIre aN essan RN ItseQE, GIFeN the uNBeNIaiQe IKJRrtaNPe the Katter has aPIuIreB RFer the nears.</i></p>

<p>Here we can see a lot more, <i>whIPh</i> can be <b>which</b>, <i>GIFeN</i> can be <b>given</b>. Also <i>ItseQE</i> can be <b>itself</b> and <i>shoulB</i> should be <b>should</b>. Now I can see the first error! </p>

<p>As <i>Nearln</i> should be the word <b>nearly</b>, <b>n</b> -> <b>y</b>. Again <i>INforKatIoN</i> should be <b>information</b>. Phrase <i>ieING alIMe</i> can translate to <b>being alive</b>, while word <i>oJINIoNs</i> is <b>opinions</b>. Now we are very close to the end, we have one more word <i>CudGemeNtal</i>, which is supposed to be <b>judgemental</b>. At this moment we have to take last few steps to correct this text, together with our knowledge of language. Here is the final message. </p>

<p><i>modern life nearly conspires against the collective. we are tormented by a relentless flow of information as well as the daily worries of an eternally insecure, unwarranted life. furthermore, we dread the thought of being alive, of sharing multiple views and opinions. as such, we are turning progressively judgemental of who we should be partnering with, on the basis that "they do not understand". in hacking, it yet implicates on the delicate subject of trust, which would require an essay on itself, given the undeniable importance the matter has acquired over the years.</i></p>

<p>Great, we broke the cipher and got the message! Thanks <a href="http://www.phrack.org/issues/69/6.html#article">Phrack</a>! </p>
