---
layout: post
title:  "RxJS Higher-order Observables"
---

![](/assets/hoo.png)

Observables that emit numbers, strings, objects or arrays are referred to as first-order observables. Higher-order observables emit an observable. There are scenarios where we need to process the results of a first observable into a second observable. For instance, consider the example below: 


{% highlight typescript %}
of(4, 1, 3)
.subscribe(x => of(x).pipe(delay(x * 1000))
    .subscribe(result => console.log(result))
);
{% endhighlight %}


In this example we have nested subscribes. We should avoid them as they can become complex and it would be difficult to unsubscribe observables. This is where high-order mapping operators come into play.

# High Order Mapping operators

High-order mapping operators map each value from an outer observable to a new inner observable, automatically subscribe to and unsubscribe from the inner observable. Ultimately higher-order mapping operators emit resulting values to the output stream.

The most commonly used operators are `concatMap`, `mergeMap` and `switchMap`

# ConcatMap


The `concatMap` operator processes items in sequence. When the outer observable emits items, `concatMap` subscribes to the first inner observable. Once it completes it emits the result to the output stream and then subscribes to the next inner observable. The process repeats until all items complete.

{% highlight typescript %}
of(4, 1, 3)
    .pipe(
        concatMap(x => of(x).pipe(delay(x * 1000)))
    .subscribe(x => console.log(x));
{% endhighlight %}

Use `concatMap` when you want to process items in sequence. For example getting data in sequence from a set of ids.

# MergeMap

The `mergeMap` operator processes items in parallel. Whenever the outer observable emits an item, `mergeMap` automatically subscribes to its inner observable. Any inner observable that completes emits the result to the output stream. Since items are processed in parallel, the result stream may not be in sequential order, but rather in the order of the inner observable completion. For instance, in the example below, `1` completes first, then `3` and then `4`.

{% highlight typescript %}
of(4, 1, 3)
    .pipe(
        mergeMap(x => of(x).pipe(delay(x * 1000))))
    .subscribe(x => console.log(x));
{% endhighlight %}

Use `mergeMap` when you want to process items in parallel, when order is not important.

# SwitchMap

The `switchMap` operator switches to the most recent item emitted. When the outer observable emits an item, `switchmap` automatically stops previous and subscribes to the new inner observable.


{% highlight typescript %}
of(4, 1, 3)
    .pipe(
        switchMap(x => of(x).pipe(delay(x * 1000))))
    .subscribe(x => console.log(x));
{% endhighlight %}

Use `switchMap` to stop previous observable and switching to the next one. For example, retrieving list data on sort change or page change.



