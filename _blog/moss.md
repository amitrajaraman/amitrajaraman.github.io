---
layout: page
title: MOSS
---

<script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-MML-AM_CHTML" async></script>

[MOSS](https://theory.stanford.edu/~aiken/moss/) (Measure Of Software Similarity), as many students might know, is an extremely powerful tool to detect plagiarism. [This paper](http://theory.stanford.edu/~aiken/publications/papers/sigmod03.pdf), which covers the ideas behind MOSS, is what I have referred to while writing this... article? I'm not quite sure what to call it.

## Introduction

If two documents were exactly the same, then it's quite easily checked. If two documents are slightly different, however, it obviously becomes more nuanced. To start off, define a $$k$$-gram as a contiguous substring of length $$k$$. Note that for some $$k$$, there are nearly as many $$k$$-grams as there are characters in the document and it is thus extremely impractical to compare every $$k$$-gram (and say consider the number of $$k$$-grams that match). A solution to this that's slightly easier on the system is to hash these $$k$$-grams and select some subset of these hashes to be the document's *fingerprints*.    
Here we assume that a "good" hash function is chosen, that is, if two fingerprints are the same then the corresponding $$k$$-grams are most probably the same as well.

A natural next question is: how do we appropriately and efficiently choose fingerprints from a sequence of hashes that guarantees the detection of (at least part of) sufficiently long matches?

We cannot do something as simple as say, choosing every $$i$$th hash value, because that does not protect against the third property described in the [subsequent section](#desirableProps). Indeed, adding a single character at the beginning of a document could result in completely different fingerprints.

An idea that comes to mind is: select some reasonable value $$w$$ (a "window width"), and ensure that you choose at least one hash value per window. Indeed, if we do this, then we can identify any matching substring of length at least $$w+k-1$$. The algorithm chooses some (non-zero) number of hashes from this window. Let the _density_ of the algorithm be the expected number of hashes it chooses.    
Now of course in real life examples, data isn't as random as that which is considered in the theoretical general case, there are often several repeating strings.

## <a name="desirableProps"></a>Desirable Properties

A couple of properties we would want in a plagiarism checker are:
* Whitespace and punctuation insensitivity.
* Noise suppression. We want the matches to be significant. For instance, two programs containing ```while``` doesn't really mean anything; there should be clear evidence of copying.
* Position independence. Switching around two paragraphs doesn't mean two programs are different.

The first issue is quite easily fixed, just preprocess the file removing all whitespace/punctuation, converting all letters to lowercase, and maybe converting all variable names to ```V```.

The second is easily remedied as well by choosing an appropriate value of $$k$$ since most boring matches are short like ```while``` or ```if```. Further, experimental studies have shown that there tends to be a sharp threshold value of $$k$$ above which results are as desired.

The third issue is slightly more involved.

## The Hash Function

Since the substrings corresponding to the hashes of two successive substrings are largely similar, it would be convenient if we had some way of easily calculating one hash value from the previous one, namely a "rolling" hash function. To do this, use the hash function used in the [Rabin-Karp Algorithm](https://en.wikipedia.org/wiki/Rabin%E2%80%93Karp_algorithm), which treats each of the "digits" in the $$k$$-gram $$c_1 \ldots c_k$$ as a number in some base $$b$$ and maps it to

$$c_1\cdot b^{k} + c_2\cdot b^{k-1} + \ldots + c_k\cdot b.$$

We can then obtain the hash value of a substring from the previous hash value by

$$H(c_2,\ldots,c_{k+1}) = (H(c_1,\ldots,c_k)-c_1\cdot b^k)\cdot b.$$

## Winnowing - which fingerprints do we choose?

Here the algorithm is described more concretely. We wish to find all substring matches between two documents such that

* there is a _guarantee threshold_ $$t$$ such that all matches of length at least $$t$$ are identified.
* there is a _noise threshold_ $$k$$ such that no strings of length less than $$k$$ are identified.

Suppose that $$k< t$$. Ideally we would choose the minimum value of $$k$$ such that there are no coincidental matches.

So coming back to our initial issue, which hashes do we choose to be representative of the document? Consider the following two methods:

* Choose the $$r$$ smallest hashes for some $$r$$. The advantage in this is that irrespective of the document size, we choose a constant number of hashes, so it is very scalable. However, it only works well when the documents are of comparable size. For example, there would probably be no matches between a large book and a paragraph taken from the book.
* Choose all hashes that are $$0\pmod p$$. This assists positional independence and gives a reasonable representative of the document. Indeed, this algorithm does work decently well. However, the problem with this is that the gap between two fingerprints we have chosen may grow unbounded, so entire paragraphs may be skipped.    
The algorithm used here, Winnowing, can be thought of as a combination of the first method and the "windows" we introduced earlier.

>**Winnowing.** Given a sequence of hashes $$h_1, h_2,\ldots, h_n$$ and a _window size_ $$w$$, winnowing chooses the hash with the minimum value in each window. If there is more than one such hash, choose the rightmost one.

For example, let `77` `74` `42` <b>`17`</b> `98` `50` <b>`17`</b> `98` <b>`8`</b> `88` `67` `77` <b>`43`</b> be the sequence of hashes and let $$w=4$$. The chosen fingerprints are marked in bold.

This ensures that a comparatively low number of fingerprints is selected (this is explained more in-depth in the next section), and also that there's no large chunks of text skipped while eliminating the issue that we raised in the beginning with something like say selecting every $$r$$th fingerprint.    
We also typically store some positional information about each fingerprint. This would be useful if we wanted to show the user where the suspected matches are.

## Density

We now consider the issue of density, which is the expected fraction of fingerprints chosen for a (random) input. We claim that the density is $$\frac{2}{w+1}$$.

Denote the set of positions of selected fingerprints by $$F$$ and let the length of the document be $$n$$. Consider the map $$C:F\to [n]$$ which maps a value $$f$$ in $$F$$ to the minimum position in $$[n]$$ which selects it as the fingerprint. More concretely,

$$C(f)=\min\{p\in[n]:\min\{H_p,\ldots,H_{p+w-1}\}=H_f\}$$

It is easily proved that $$C$$ is monotonic increasing. To show our claim, we first assume that the space of hash values is large, so there is exactly one minimum value for each window. Let $$X_i$$ be a random variable that is $$1$$ if $$i=C(f)$$ for some $$f\in F$$. The density is then equal to $$\frac{\sum_{i=1}^n E[X_i]}{n}$$ (the expectation of the sum of random variables is equal to the sum of their expectations).

For some $$i>1$$, consider $$W_i$$ and $$W_{i-1}$$. Let $$p$$ be the position of the minimum in the union of these two windows (which spans $$w+1$$ positions).

* If $$p=i-1$$, then since $$p$$ is not in $$W_i$$, $$i$$ must lie in the image of $$C$$ and $$X_i=1$$.
* If $$p=i+w-1$$, then $$i$$ must be in $$\Im(C)$$ and $$X_i=1$$.
* Finally, if $$p\in [i,i+w-2]\cap\mathbb{N}$$, then as $$p$$ is in both $$W_{i-1}$$ and $$W_i$$, $$X_i=0$$.

For input drawn from a uniformly random distribution, the probability of $$p$$ being $$i-1$$ or $$i+w-1$$ is $$\frac{2}{w+1}$$ (because $$p$$ can take $$w+1$$ possible values). That is, $$E[X_i]=\frac{2}{w+1}$$. Note that as this value is independent of the position $$i$$ (asymptotically, we can disregard the case where $$n-i< w$$), the density is just

$$d=\frac{2}{w+1}$$

