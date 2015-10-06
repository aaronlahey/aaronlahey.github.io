---
layout: post
title:  "Insertion Sort"
date:   2015-10-05
categories: algorithms
---

In my [previous post](http://blog.aaronlahey.com/algorithms/2015/10/04/selection-sort.html) I gave a brief introduction to the selection sort algorithm. Today, I'd like to look at another elementary sorting algorithm called insertion sort. Insertion sort is interesting because the performance characteristics of the algorithm directly depend on the state of the input. In the best case, when given pre-sorted input, the algorithm performs `N-1` compares and `0` swaps. However, in the worst case, when the input is in reverse-sorted order, it performs `N^2/2` compares and `N^2/2` swaps.

Insertion sort starts at the first index in the array and progresses to the last index in the array. If the current element is less than the element directly to its left they are swapped. This process is repeated until the element to its left is less than the current element. After the element has been moved into sorted order, all elements before our current element are guaranteed to be in sorted order and we can move on to the next. Consider the following example...

{% highlight ruby %}
[5, 3, 8, 9, 1]
{% endhighlight %}

First, we would start at index `0`. There are no elements to its left so we don't need to move it yet.

{% highlight ruby %}
[5, 3, 8, 9, 1] #=> [5, 3, 8, 9, 1]
 ^
{% endhighlight %}

Moving on to index `1` we would compare it to the element to its left. In this case, `3` is less than `5` so we must swap these two elements.

{% highlight ruby %}
[5, 3, 8, 9, 1] #=> [3, 5, 8, 9, 1]
 ^  ^
{% endhighlight %}

Next, we would inspect index `2`, compare `8` to `5`, and conclude no work needs to be done.

{% highlight ruby %}
[3, 5, 8, 9, 1] #=> [3, 5, 8, 9, 1]
       ^
{% endhighlight %}

After inspecting index `3`, we would compare `9` to `8`, and again do nothing.

{% highlight ruby %}
[3, 5, 8, 9, 1] #=> [3, 5, 8, 9, 1]
          ^
{% endhighlight %}

After reaching index `4`, we would compare `1` to `9`, swap them, `1` to `8`, swap them, `1` to `5`, swap them, `1` to `3`, swap them, and leave `1` as the first element in the array.

{% highlight ruby %}
[3, 5, 8, 9, 1] #=> [3, 5, 8, 1, 9]
          ^  ^
[3, 5, 8, 1, 9] #=> [3, 5, 1, 8, 9]
       ^  ^
[3, 5, 1, 8, 9] #=> [3, 1, 5, 8, 9]
    ^  ^
[3, 1, 5, 8, 9] #=> [1, 3, 5, 8, 9]
 ^  ^
{% endhighlight %}

Steps `3`, `4`, and `5` should help illuminate why this particular algorithm's performance is so closely tied to the order of its input. Steps `3` and `4` required only a single compare to decide that the current element was already in sorted order and that no extra work was required. However, in step `5`, the last element was in the worst possible position and required a comparison and a swap with every other element in the array.

The implementation for insertion sort is even more compact than that of selection sort, requiring only two simple `for` loops...

{% highlight java linenos %}
public class InsertionSort<T extends Comparable<T>> {
    public void sort(T[] elements) {
        int length = elements.length;

        // for every element in the array...
        for(int i = 0; i < length; i++) {
            // as long as we're not at position 0 and the element to the left is
            // smaller than our current element...
            for(int j = i; j > 0 && isLessThan(elements, j, j - 1); j--) {
                // ...swap the current element and the previous one
                swap(elements, j, j - 1);
            }
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
