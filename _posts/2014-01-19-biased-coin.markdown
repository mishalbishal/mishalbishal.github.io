---
layout: post
title:  "Biased Coin"
date:   2014-01-19 00:00:00 -0400
---
Note: This post was imported from a previous blog site.

This problem has come up often recently: if you have an biased coin that lands on heads more than it does on tails, can you use that coin in order to simulate a fair coin? So basically, use the coin and achieve the same effect as if you had a fair coin.

The first time I saw this question was during a mock technology interview that my school's Office of Career Services provided. Then it showed up in class today, as I was shopping an Algorithms class (CS124). How can you possibly answer this question? Why not give it a go before reading the next couple of paragraphs?

The coin has a bias and we can say that this bias is `P`, so the probability of landing on heads is `P`. The probability of landing on tails is `(1-P)`. Now I had originally thought of trying to find `P` and then compensate based on `P`'s value in order to get a fair way of using this coin. However, there is another way without actually figuring out what the value of `P` is:

Instead of just tossing the biased coin once, toss it more than one times (here I'll do two tosses). Here's the probabilities of the 4 possible outcomes for two tosses:

* Heads, then Tails -> `P*(1-P)`. *The probability of this is `P*(1-P)`. This is because the probability of flipping heads is `P` and the probability of flipping tails is `(1-P)`, so overall the probability of the two is `P times (1-P)`.*
* Tails, Head -> `(1-P)*(P)`
* Head, Head -> `(P)*(P)`
* Tail, Tail -> `(T)*(T)`

Notice how the probability of the 1st and 2nd outcome is the same! `P*(1-P)` is the same as `(1-P)*P`. Because the probability of getting Head then Tail is the same as getting Tail and then Head, you can use this equal probability in order to simulate a fair coin. Let the Head, Tail case be called "Heads" and the Tail, Head case be called "Tails". The other two outcomes 3. Head, Head and 4. Tail, Tail can be ignored.

So in order to simulate a fair coin, flip the unbiased coin twice. If you get the Head,Tail pair then consider that as "Heads" and if you get the Tail,Head pair consider that "Tails." If you don't get either of these two, repeat until you do. And there you have it! A way of getting fair outcomes from a biased coin.

From the interviewer, I learned how to prove that this method worked to provide a fair coin simulator. In class, there was a discussion of improvements to this method. I won't talk about those things here, but still isn't it cool how you can solve the problem of the biased coin in this way?