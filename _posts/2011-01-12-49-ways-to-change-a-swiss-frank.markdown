---
layout: post
title: "There are 49 ways to change a swiss frank"
date: 2011-01-12 02:37:00
tags:
  - scheme
  - sicp
---

...and 292 ways to change a US dollar

{% highlight scheme %}
(define (count-change amount) (cc amount 4))

(define (cc amount kinds-of-coins)
    (cond
        ((= amount 0) 1)
        ((or (< amount 0) (= kinds-of-coins 0)) 0)
        (else
            (+
                (cc amount (- kinds-of-coins 1))
                (cc 
                    (- amount (first-denomination kinds-of-coins)) 
                    kinds-of-coins)))))

(define (first-denomination kinds-of-coins)
    (cond
        ((= kinds-of-coins 1) 5)
        ((= kinds-of-coins 2) 10)
        ((= kinds-of-coins 3) 20)
        ((= kinds-of-coins 4) 50)))


(count-change 100)
{% endhighlight %}
