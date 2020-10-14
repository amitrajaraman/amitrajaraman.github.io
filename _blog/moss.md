---
layout: page
title: MOSS
---

<script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-MML-AM_CHTML" async></script>

[MOSS](https://theory.stanford.edu/~aiken/moss/) (Measure Of Software Similarity), as many college students might know, is an extremely powerful tool to detect plagiarism. [This paper](http://theory.stanford.edu/~aiken/publications/papers/sigmod03.pdf), which covers the ideas behind MOSS, is what I have referred to while writing this... article? I'm not quite sure what to call it.

## Introduction

If two documents were exactly the same, then it's quite easily checked. If two documents are slightly different, however, it becomes slightly more nuanced. To start off, define a $$k$$-gram as a contiguous substring of length $$k$$. Note that for some $$k$$, there are nearly as many $$k$$-grams as there are characters in the document and it is thus extremely impractical to compare every $$k$$-gram (and say consider the number of $$k$$-grams that match). A solution to this that's slightly easier on the system is to hash these $$k$$-grams and select some subset of these hashes to be the document's *fingerprints*.    
Here we assume that a "good" hash function is chosen, that is, if two fingerprints are the same then the corresponding $$k$$-grams are most probably the same as well.

A popular approach to select this subset is to just choose those hashes that are $$0 \pmod p$$. However, this comes with its own issues. Since the gap between two fingerprints we have chosen may grow unbounded, we may not detect many matches.    
A natural next step is: how do we appropriately and efficiently choose fingerprints from a sequence of hashes that guarantees the detection of (at least part of) sufficiently long matches?