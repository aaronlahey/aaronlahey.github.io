---
layout: post
title:  "Merge Sort"
date:   2015-11-06
categories: algorithms
---

In previous posts I gave a brief introduction to [selection sort](http://blog.aaronlahey.com/algorithms/2015/10/04/selection-sort.html) and [insertion sort](http://blog.aaronlahey.com/algorithms/2015/10/05/insertion-sort.html). These simple sorting algorithms provide a nice introduction to implementing algorithms, time and space complexity, as well as slightly lower-level array operations. These algorithms, particularily insertion sort, aren't just simple versions of "real" sorting algorithms, they're actually used in real production code. For example, if we look at the Java `Arrays.sort` method for `Object[]`...

{% highlight java %}
public static void sort(Object[] a) {
    if (LegacyMergeSort.userRequested)
        legacyMergeSort(a);
    else
        ComparableTimSort.sort(a, 0, a.length, null, 0, 0);
}
{% endhighlight %}

...we can see when the user has not explicitly requested the legacy merge sort behavior Java will use a version of the timsort algorithm. The very first comment in `ComparableTimSort.java` states that it is a "near duplicate" of `TimSort.java`. The opening comments to `TimSort.java` have this to say...

{% highlight java %}
/*
 * While the API to this class consists solely of static methods, it is
 * (privately) instantiable; a TimSort instance holds the state of an ongoing
 * sort, assuming the input array is large enough to warrant the full-blown
 * TimSort. Small arrays are sorted in place, using a binary insertion sort.
 */
{% endhighlight %}

There it is in black and white. When the array is small enough Java will use our old friend, insertion sort.

Even merge sort, the algorithm I'll focus on in this post, is a part of the Java language. Not only is it available as the `Arrays.sort` implementation when specifying `-Djava.util.Arrays.useLegacyMergeSort=true` at runtime, but it is also a contributing idea to timsort's implementation according to the source of all truth, Wikipedia:

>Timsort is a hybrid stable sorting algorithm, derived from merge sort and insertion sort, designed to perform well on many kinds of real-world data.

## Implementation

Merge sort is a recursive algorithm. When given two sorted arrays, it merges them into a single sorted array. Instead of using two different sorted arrays that will be merged into the final array, the implementation we discuss today will use a single auxiliary array which acts as two logical arrays.

The merge operation is fundamental to this algorithm. For this operation to function correctly a very important precondition must be satisfied...each logical array being merged must be in sorted order. To merge, we must keep track of the index of the first source array (`i`), the index of the second source array (`j`), and the current index of the destination array (`k`). The merge, we must first copy all elements from the array being sorted into an auxiliary array. After that, we compare the values at index `i` and index `j`, put the minimum of those two values into the destination array at index `k`, and increment both the index of the source array used and the index of the destination array.

{% highlight java linenos %}
public void merge(T[] elements, T[] aux, int lo, int mid, int hi) {
    // First, copy the elements from the array being sorted into an
    // auxiliary array. We use System.arraycopy instead of a manual
    // for loop for performance reasons. We also only need to copy
    // from the `lo` index up to the number of elements in the subarray.
    // This can be calculated by using hi - lo + 1.
    System.arraycopy(elements, lo, aux, lo, hi - lo + 1);

    // The start of the first source array is the start of the
    // auxiliary array.
    int i = lo;

    // The start of the second source array is the index after the
    // middle of the auxiliary array.
    int j = mid + 1;

    // The auxiliary array is the exact same size as the destination
    // array so we can iterate over every value of `k` and know it
    // will include every value of `i` and `j`. We need to iterate
    // while `k` is less than or equal to `hi` because `hi` is the last
    // index of the current array, not the length.
    for(int k = lo; k <= hi; k++) {

        // Here we check to see if we've consumed every value from
        // the first array. We know we have when the current index
        // of `i` is the starting value of `j`.
        if(i > mid) {
            // If we have consumed every value from the first array
            // we can continually pull from the second array and
            // increment its index.
            elements[k] = aux[j++];

        // Here we check to see if we've consumed every value from
        // the second array. We know we have when the current index
        // of `j` is higher than than `hi`. `hi` in this case will be
        // the last index of the current subarray.
        } else if(j > hi) {
            // If we have consumed every value from the second array
            // we can continually pull from the first array and
            // increment its index.
            elements[k] = aux[i++];

        // This branch is the first comparison between the two arrays.
        // If the current element of the first array is smaller than
        // the current element of the second we'll place that value
        // at the current index of the destination array and increment
        // the source array's index.
        } else if(aux[i].compareTo(aux[j]) < 0) {
            elements[k] = aux[i++];

        // If the current element of the first array is equal or larger
        // than the current element of the second, we'll use the second
        // array's value.
        } else {
            elements[k] = aux[j++];
        }
    }
}
{% endhighlight %}

The next method we'll look at is a variation on our public interface. This method will enable us to implement a recursive algorithm that utilizes a single auxiliary array. It will take four arguments: the array being sorted, the auxiliary array, the start index of the array (`lo`) and the end index of the array (`hi`). Its job is simply to divide the array into two halves by calculating the midpoint between the current `lo` and `hi` values, sorting the two resulting logical subarrays, and then merging them.

{% highlight java linenos %}
// The four arguments here, as mentioned above, consist of the array
// being sorted, the auxiliary array, the current start index, and
// the current end index. Remember this method is recursively called
// so the `lo` value will not always be 0 and the `hi` value will
// not always be `elements.length - 1`
public void sort(T[] elements, T[] aux, int lo, int hi) {
    // Here we check to see if our current logical array is empty. A
    // logical array is considered empty if the the end index is
    // greater than or equal to the start index. For example, if our
    // subarray starts at index 4 and ends at index 4 it effectively
    // an empty array. If the array is, in fact, empty, we have no
    // work to do and can simply return.
    if (hi <= lo) { return; }

    // Here we calculate the middle of the logical array being sorted.
    // If the current `lo` value is 4 and the current `end` value is
    // 7, the mid point is 4 + (7 - 4) / 2, or 5.
    int mid = lo + (hi - lo) / 2;

    // Given the mid point we can sort the first logical array which,
    // in the example above, would be an array from index 4 to 5.
    sort(elements, aux, lo, mid);

    // Next we sort the second logical array, which in the example above,
    // would be a subarray from index 6 to 7.
    sort(elements, aux, mid + 1, hi);

    // After those two subarrays are sorted, we can merge them using
    // the merge method described above. Once we've merged a sorted
    // array from 4 to 5, and a sorted array from 6 to 7, we know
    // that the array from 4 to 7 is sorted.
    merge(elements, aux, lo, mid, hi);
}
{% endhighlight %}

Finally, all that's left is our public interface which is a single public void sort method that takes an array of `Comparable` objects.

{% highlight java linenos %}
@Override
@SuppressWarnings("unchecked cast")
public void sort(T[] elements) {
    // We want to create the auxiliary array in the public sort method
    // so it can be reused. This lets us avoid a potential explosion of
    // memory use when creating new auxiliary on every recursive sort call.
    T[] aux = (T[]) new Comparable[elements.length];

    // Finally, we delegate to our recursive version of the sort method
    // passing in the elements, the auxiliary array, the start index which
    // will always be 0, and the last index of the array to be sorted which
    // will always be the length - 1.
    sort(elements, aux, 0, elements.length - 1);
}
{% endhighlight %}

Merge sort is an interesting algorithm. The ideas it uses are the basis for more complex algorithms such as timsort and it was the default `Arrays.sort` algorithm prior to Java 7. Now that we've covered three different sorting algorithms, I encourage you to start looking through the JDK source code. See if you can recognize similiar patterns to the three sorting algorithms we've discussed thus far. Be on the lookout for my next post on quick sort! I'll leave you with the full merge sort implementation without comments.

{% highlight java linenos %}
public class MergeSort<T extends Comparable<T>> implements SortingAlgorithm<T> {
    @Override
    @SuppressWarnings("unchecked cast")
    public void sort(T[] elements) {
        T[] aux = (T[]) new Comparable[elements.length];
        sort(elements, aux, 0, elements.length - 1);
    }

    public void sort(T[] elements, T[] aux, int lo, int hi) {
        if (hi <= lo) { return; }
        int mid = lo + (hi - lo) / 2;
        sort(elements, aux, lo, mid);
        sort(elements, aux, mid + 1, hi);
        merge(elements, aux, lo, mid, hi);
    }

    public void merge(T[] elements, T[] aux, int lo, int mid, int hi) {
        System.arraycopy(elements, lo, aux, lo, hi - lo + 1);

        int i = lo;
        int j = mid + 1;
        for(int k = lo; k <= hi; k++) {
            if(i > mid) {
                elements[k] = aux[j++];
            } else if(j > hi) {
                elements[k] = aux[i++];
            } else if(aux[i].compareTo(aux[j]) < 0) {
                elements[k] = aux[i++];
            } else {
                elements[k] = aux[j++];
            }
        }
    }
}
{% endhighlight %}
