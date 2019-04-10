---
layout: post
title: "dplyr's if_else vs base R's ifelse"
---

![](https://raw.githubusercontent.com/JavOrraca/Home/gh-pages/assets/img/dplyr.jpg)

When working with data sets of significant size, I'm always on the lookout for more efficient code (not necessarily fewer lines of code, but code that processes data faster and more effectively). Recently, I was getting antsy waiting for a simple if/else statement to run via R (I was working with a large GB-sized file on my personal computer). While waiting, I researched alternative approaches for performance improvement and found that [dplyr](https://dplyr.tidyverse.org/), part of R's Tidyverse and one of my favorite packages for data manipulation, has a vectorized if function perfect for the occasion.

I was quite surprised to find that base R's ifelse function ran over 70% slower than dplyr's if_else on my data set. This is due to dplyr's if_else function being a vectorized interpretation of base R's ifelse, meaning that dplyr's if_else function works not just on a single value at a time but on the whole vector of values simulatneously. Sometimes, your problem can't utilize vectorized functions, but for if/else cases on massive data sets, this function is a gem! 

Sources:
* [dplyr: Vectorised if](https://dplyr.tidyverse.org/reference/if_else.html)
* [Tidyverse: A Collection of R Packages](https://www.tidyverse.org/)