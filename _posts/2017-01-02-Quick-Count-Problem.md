---
layout: post
title:  "Quick Count Problem"
date:   2017-01-02
categories: [R, Python]
tags: [R, Python]
---


A friend of mine who works for a Environmental Science consultancy approached me with an data analysis problem. She was working on building a park in Sydney that had a wetland feature and were monitoring water flow via a pipe to the site. She had flow sampler running that turned off and on periodically and she needed to know how many times the pump turned on and off during a given period. The an an anomlously high reading in one of the metrics in the pump logs indicated a switch on or off. My friend and a colleague were manually going through the logs and marking a switch on based on the values of this metric. I told her this could be done fairly easily programmatically.



Lets take a look at the data set:


{% highlight text %}
##    ID     Time Event             Date Minutes
## 1 448 42361.73     1 23/12/2015 17:27  149.34
## 2 449 42361.73     0 23/12/2015 17:28    1.33
## 3 450 42361.73     1 23/12/2015 17:29    1.33
## 4 451 42361.73     0 23/12/2015 17:31    1.34
## 5 452 42361.73     1 23/12/2015 17:32    1.34
## 6 453 42361.73     0 23/12/2015 17:33    1.35
{% endhighlight %}

The varible of interest here is 'Minutes' an anomolously high value occurs when the flow sampler turns on. Lets look at the distribution of values for the Minutes variable. 


{% highlight r %}
library(ggplot2)
ggplot(flow, aes(x = Minutes)) +
  geom_density() +
  scale_x_continuous(limits = c(0, 250)) +
  scale_y_sqrt()
{% endhighlight %}

![](/assets/unnamed-chunk-2-1.png)

A quick scan of the data set and this plot shows that values usuall fall under 2 with a sampler on value being significantly higher usually above 100. We can loop through our dataset and count the rows between our high number and reset our counter when we encounter one of these large values.
The large values are atlease 1 order of magnitude higher than our other 'Minutes' values so being less than the log of our maximum minutes value should give us a criteria to reset the counter.


{% highlight r %}
counter <- 0
for (i in 1:length(flow$Minutes)){
  
   if(flow$Minutes[i] <= log(max(flow$Minutes))){
     counter <- counter + 1
     sampler <- "Off"
   } else {
     counter <- 0
     sampler <- "On"
   }
     flow$count[i] <- counter
     flow$sampler[i] <- sampler
}
{% endhighlight %}

Let's take a look at our data again.


{% highlight r %}
flow[c(30:40),]
{% endhighlight %}



{% highlight text %}
##     ID     Time Event             Date Minutes count sampler
## 30 477 42361.76     0 23/12/2015 18:07    2.74    29     Off
## 31 478 42361.76     1 23/12/2015 18:09    1.37    30     Off
## 32 479 42361.76     0 23/12/2015 18:10    1.38    31     Off
## 33 480 42361.76     1 23/12/2015 18:11    1.38    32     Off
## 34 481 42361.76     0 23/12/2015 18:13    1.38    33     Off
## 35 482 42361.85     1 23/12/2015 20:31  137.89     0      On
## 36 483 42361.86     0 23/12/2015 20:32    1.34     1     Off
## 37 484 42361.86     1 23/12/2015 20:33    1.34     2     Off
## 38 485 42361.86     0 23/12/2015 20:35    1.35     3     Off
## 39 486 42361.86     1 23/12/2015 20:36    1.35     4     Off
## 40 487 42361.86     0 23/12/2015 20:37    1.35     5     Off
{% endhighlight %}


Looks like the counter is working. Wherever the counter resets the flow sampler is turning on. We can double check that its working for all our values by looking at the plot this time differentiating between sample On and sampler Off.

![](/assets/unnamed-chunk-4-1.png)

Done. This could be improved by dynamically determing the threshold rather than explicitly defining one. 

