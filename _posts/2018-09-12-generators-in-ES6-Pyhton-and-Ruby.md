---
layout: post
title:  "Making a Fibonator with Generators in ES6, Python and Ruby"
---
Fibonacci numbers are beautiful, this sequence appears in various forms, from [data structures][fib_heap] to [sunflower seeds][sunflower_seeds]. Let's see how we can make a Fibonator - a generator which returns Fibonacci sequence.

With ES6, JavaScript finally got generators, while in Python they have been available long time ago. In Python, yield keyword converts function to generator:

{% highlight python %}
import itertools
def make_fibonator():
    a = b = 1
    while True:
        yield a
        a, b = b, a + b

fibonator1 = make_fibonator()
print (next(fibonator1))
print (next(fibonator1))
print (next(fibonator1))
#prints 1 1 2

fibonator2 = make_fibonator()
for n in itertools.takewhile(lambda x: x < 100, fibonator2):
    print(n)
#prints 1 1 2 3 5 8 13 21 34 55 89
{% endhighlight %}

In ES6, yield keyword is not enough, function keyword has to be followed by asterisk (function*) to make function a generator:

{% highlight javascript %}
function* make_fibonator() {
    let [a, b] = [1, 1];
    while (true) {
        yield a;
        [a, b] = [b, a + b];
    }
}
const fibonator1 = make_fibonator();
console.log(fibonator1.next().value);     // 1
console.log(fibonator1.next().value);     // 1
console.log(fibonator1.next().value);     // 2
console.log(fibonator1.next().value);     // 3

const fibonator2 = make_fibonator();

let count = 0
for (let value of fibonator2) {
    console.log(value);
    count++;
    if (count >= 10) break;
} 

// Prints 1 1 2 3 5 8 13 21 34 55
{% endhighlight %}

Ruby also have yield keyword, but it means something different than in ES6 and Python. In Ruby, yield calls a code block associated with the method which contains that yield. Here is silly example:
{% highlight ruby %}
def three_times_yield 
    yield 1
    yield 2
    yield 3
end

three_times_yield {|n| print n, " "}
puts
#Prints 1, 2, 3
{% endhighlight %}

There is no encoded state machine here, we cannot print one number, stop and then resume, so clearly this is not a generator. To get generator in Ruby we need to use [Enumerator][Enumerator] class, where yielder (which is instance of Enumerator::Yielder) is given as block parameter. Now we can call yield method (do not confuse with yield keyword). Here is fibonator using Enumerator:

{% highlight ruby %}
def make_fibonator
    fibonator = Enumerator.new do |yielder|
        a = b = 1
        loop do 
            yielder.yield(a)
            a, b = b, a + b 
        end
    end
    return fibonator
end
{% endhighlight %}

Confusing part is yield method, which share the name with yield keyword. Alternitavley, [Enumerator][Enumerator] class provide << method, so we can replace line "yielder.yield(a)" with "yielder << a", bit I find this even more confusing, so I just stick with yield method. Internally, state machine is implemented using [Fibers][Fibers].

Now, once we have fibonator, we can call next method to print sequence manually

{% highlight ruby %}
fibonator1 = make_fibonator
p fibonator1.next
p fibonator1.next
p fibonator1.next
#prints 1,1,2

count = 0
loop  { print fibonator1.next, " "; count += 1; break if count >= 5 }
puts
#prints 3 5 8 13 21
{% endhighlight %}

Or using iterator methods. After finishing, these methods reset generator state back to initial state (as we expect):

{% highlight ruby %}
fibonator2 = make_fibonator

count = 0
fibonator2.each { |n| print n, " "; count += 1; break if count >= 5 }
puts
#prints [1, 1, 2, 3, 5]
p fibonator2.first(10)
#prints [1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
{% endhighlight %}

Using iteration methods, it is easy to make endless loop. For example, let's say we want to find first 10 Fibonacci sequence numbers which divides by 5:
{% highlight ruby %}
fibonator2.select{|n| n % 5 == 0}.first(10)
{% endhighlight %}

This will never return, as select will happily draw number from fibonator, which produces infinite sequence. To avoid this, we need a lazy iteration method.
{% highlight ruby %}
p fibonator2.lazy.select{|n| n % 5 == 0}.first(10)
#prints [5, 55, 610, 6765, 75025, 832040, 9227465, 102334155, 1134903170, 12586269025]
{% endhighlight %}

Lazy enumeration forces first method to draw numbers, so in this case sequence works as expected. There is a nice overview about lazy Ruby features in this [blog post][lazy_ruby].  


[fib_heap]: https://en.wikipedia.org/wiki/Fibonacci_heap
[lazy_ruby]: http://patshaughnessy.net/2013/4/3/ruby-2-0-works-hard-so-you-can-be-lazy
[sunflower_seeds]: https://momath.org/home/fibonacci-numbers-of-sunflower-seed-spirals/
[Enumerator]: https://ruby-doc.org/core-2.2.0/Enumerator.html
[Fibers]: http://ruby-doc.org/core-2.2.0/Fiber.html