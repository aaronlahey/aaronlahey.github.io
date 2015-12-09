---
layout: post
title: "Selection Sort"
date: 2015-10-04
categories: algorithms
---

Having majored in music instead of computer science, my knowledge of algorithms and data structures is an area that's ripe for improvement. Over the next few blog posts I'll describe a series of different algorithms and data structures, their implementation, their performance characteristics, and some of their possible uses.

Selection sort is an elementary sorting algorithm. It's simple to implement and serves as a great introduction to sorting algorithm implementations. Starting from the beginning of the array it scans each element keeping track of the smallest one. After locating the index of the smallest element, it swaps the first element and the smallest element. Then, it moves to the second index and repeats the same process again. For this array...

{% highlight ruby %}
[5, 3, 8, 9, 1]
{% endhighlight %}

...we would start at index `0` and assume that `5` is the smallest value. After observing index `1` we would then see `3` is now the smallest value. `1` would remain the index with the smallest value until we observed index `4` and arrive at the end of the array. At this point, we would swap index `0` with index `4`. The following array would result...

{% highlight ruby %}
[5, 3, 8, 9, 1] #=> [1, 3, 8, 9, 5]
 ^           ^
{% endhighlight %}

Now, we'll move to index `1` and repeat the same process of observing all following elements, remembering that index, and swapping it with `1`. In this case, index `1` *is* the smallest so we'll swap `1` with `1` and move on...

{% highlight ruby %}
[1, 3, 8, 9, 5] #=> [1, 3, 8, 9, 5]
    ^
{% endhighlight %}

Next, starting on index `2` and observing indicies `3` and `4`, we would notice `4` is the smallest and swap `2` and `4`...

{% highlight ruby %}
[1, 2, 8, 9, 5] #=> [1, 3, 5, 9, 8]
       ^     ^
{% endhighlight %}

Next, starting on index `3`, we would compare that value to the value at `4` and swap them...

{% highlight ruby %}
[1, 3, 5, 9, 8] #=> [1, 3, 5, 8, 9]
          ^  ^
{% endhighlight %}

Finally, because index `4` is the last (and consequently the smallest), we swap index `4` with itself and end up with a sorted array...

{% highlight ruby %}
[1, 3, 5, 8, 9] #=> [1, 3, 5, 8, 9]
             ^
{% endhighlight %}

Thinking about the performance characteristics of selection sort we can see that in every case, regardless of whether the array is pre-sorted or not, we will perform `N` swaps and `N^2/2`. If we were to observe every element in the array *for* every element in the array we would perform `5 * 5` or `5^2` elements. However, we do not observe elements that are already assumed to be in sorted order.

Sorting algorithms in Java utilize the `Comparable<T>` interface. Implementations of this interface define what it means to compare themselves to an object of type `T`. When the receiver is deemed to be "less than" the argument the method should return `-1`, if they are the same, `0`, and if greater than, `1`...

{% highlight java linenos %}
Integer x = 42;
Integer y = 0;

x.compareTo(y) // 1
y.compareTo(x) // -1
x.compareTo(x) // 0
y.compareTo(y) // 0
{% endhighlight %}

Here is a simple implementation of selection sort in Java...

{% highlight java linenos %}
public class SelectionSort<T extends Comparable<T>> {
    public void sort(T[] elements) {
        int length = elements.length;

        // start at index 0 and iterate through each element
        for(int i = 0; i < length; i++) {
            int min = i;

            // start at our current index and look for the smallest element
            for(int j = i; j < length; j++) {
                // when the current element is smaller than the known smallest element...
                if(isLessThan(elements, j, min)) {
                    // then remember that element as the smallest
                    min = j;
                }
            }

            // after observing every element, swap the current element and
            // the known smallest element.
            swap(elements, min, i);
        }
    }

    private boolean isLessThan(T[] elements, int i, int j) {
        return elements[i].compareTo(elements[j]) < 0;
    }

    private void swap(T[] elements, int i, int j) {
        T swap = elements[i];
        elements[i] = elements[j];
        elements[j] = swap;
    }
}
{% endhighlight %}
