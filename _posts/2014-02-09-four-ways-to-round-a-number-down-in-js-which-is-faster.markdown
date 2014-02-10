---
layout: post
title:  "Four ways to round a number down in JS: Which is faster?"
date:   2014-02-09 22:28:49
description: "There are some operations in JavaScript that can be accomplished multiple ways.  One such task is the simple act of rounding a positive number down to its nearest integer."
---

There are some operations in JavaScript that can be accomplished multiple ways.  One such task is the simple act of rounding a positive number down to its nearest integer.  I've found four different ways this can be done in JavaScript, and was recently curious which is the best (read: fastest) way.

#The competitors
* * *

## Math.floor

I imagine this is probably the method most people use to round a number down because when dealing with numbers and division and math, it seems natural to use the `Math` library to round down to the integer floor.
<br/>
<br/>
Usage:
{% highlight javascript %}
var num = 5.5;
Math.floor(num);  // 5
{% endhighlight %}

## parseInt

Another rather intuitive way to round a positive decimal number down to an integer value is to literally parse the decimal number as an integer.  You have to specify a radix parameter as the second argument to `parseInt` indicating what base you intend to use when parsing the integer.  A radix of 10 converts the first argument into a base 10 integer.
<br/>
<br/>
Note: I emphasize that this can be used to round a _positive_ decimal number down to an integer value because this method, along with the subsequent methods, essentially just lops off the numbers after the decimal point so they only really round _down_ with a positive number input.  By doing this, they will actually round a negative number _up_.
<br/>
<br/>
Usage:
{% highlight javascript %}
var num = 5.5;
parseInt(num,10);  // 5
{% endhighlight %}

## Double bitwise NOT (~~)

Okay, I don't see many people using the `~~` in their code, but I figured it's worth a mention.  As mentioned [here](http://james.padolsey.com/javascript/double-bitwise-not/), applying the double bitwise NOT operator (~~) to a decimal number has the effect of rounding it down, though people not familiar with this method will certainly consider it less readable.
<br/>
<br/>
Usage:
{% highlight javascript %}
var num = 5.5;
~~num;  // 5
{% endhighlight %}

## Bitshift by zero bits

I definitely have not seen this particular method out in the wild, but was curious to see how it stacked up speed-wise.  It entails bitshifting the input, but specifying zero for the amount of bits to be shifted.  When a bitshifting operation takes place, decimal values are lost.  So 'bitshifting by zero places' doesn't do much besides just losing the numbers to the right of the decimal.
<br/>
<br/>
Usage:
{% highlight javascript %}
var num = 5.5;
num>>0;  // 5
   // OR
num<<0;  // 5
{% endhighlight %}
<br/>
* * *
# My test

To determine which of these methods is the fastest, my test involved looping 2 billion times, rounding a positive decimal down at each iteration.
<br/>
<br/>
{% highlight javascript %}
// Testing Math.floor
var start = Date.now();
for (var i = 0.5; i<=2000000000 ; i++){
  Math.floor(i);
}
var end = Date.now();
console.log('Math.floor elapsed time: ',(end - start),' ms');


// Testing parseInt
start = Date.now();
for (var i = 0.5; i<=2000000000 ; i++){
  parseInt(i,10);
}
end = Date.now();
console.log('parseInt elapsed time: ',(end - start),' ms');


// Testing double bitwise NOT
start = Date.now();
for (var i = 0.5; i<=2000000000 ; i++){
  ~~(i);
}
end = Date.now();
console.log('~~ elapsed time: ',(end - start),' ms');


// Testing bitshift right zero places
start = Date.now();
for (var i = 0.5; i<=2000000000 ; i++){
  i>>0;
}
end = Date.now();
console.log('>>0 elapsed time: ',(end - start),' ms');


// Testing bitshift left zero places
start = Date.now();
for (var i = 0.5; i<=2000000000 ; i++){
  i<<0;
}
end = Date.now();
console.log('<<0 elapsed time: ',(end - start),' ms');
{% endhighlight %}

# Results

For reference, I ran this on a 2013 Macbook Air with a 1.7 GHz Intel Core i7 and 8 GB 1600 MHz DDR3 memory.  My results were:
<br/>
<br/>
{% highlight javascript %}
Math.floor elapsed time:  2552  ms
parseInt elapsed time:  260851  ms
~~ elapsed time:  2498  ms
>>0 elapsed time:  2497  ms
<<0 elapsed time:  2537  ms
{% endhighlight %}
<br/>
Yes.  You read that right.  `parseInt` took over _4 minutes and 20 seconds_ to complete this task, while the other methods took less than 3 _seconds_.  Oof!  I tried another test, reducing the number of iterations to 1 billion.
<br/>
<br/>
{% highlight javascript %}
Math.floor elapsed time:  1288  ms
parseInt elapsed time:  9952  ms
~~ elapsed time:  1246  ms
>>0 elapsed time:  1248  ms
<<0 elapsed time:  1249  ms
{% endhighlight %}
<br/>
Again, `parseInt` took much longer than the others to complete, but interestingly, even though halving the number of iterations pretty much halved the elapsed time for every other method, `parseInt` completed in about 1/25th the time as before.  Huh?
<br/>
<br/>
Irrespective of the strangely non-linear elapsed times of `parseInt`, it proved to be the _worst_ method for achieving this simple task.  Not surprisingly, the bitwise manipulations seemed to be the fastest with `Math.floor` not far behind.  To dig a little deeper, I ran the '1 billion iterations' test in a loop 20 times back-to-back and calculated the average elapsed time for each of the methods (except for `parseInt` because... well, I didn't want to wait till next week to find out the results!)
<br/>
<br/>
{% highlight javascript %}
Math.floor average elapsed time (n = 20):  1193.7 ms
~~ average elapsed time (n = 20):  1197.15 ms
>>0 average elapsed time (n = 20):  1193.95 ms
<<0 average elapsed time (n = 20):  1193.7 ms
{% endhighlight %}
<br/>
So ultimately, it seems that `Math.floor` actually performs on-par with the bitwise methods, and of course, using `Math.floor` makes the code much more readable to others than using bitwise manipulations.
<br/>
<br/>
Takeaways from this experiment:  `Math.floor` wins on the combination of speed + clarity, while `parseInt` almost made my computer burst into flames...
