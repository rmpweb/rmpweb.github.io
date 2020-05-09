---
layout: post
title:  "Unsubscribe from RxJS Observables"
---

![](/assets/rxjsng.png)

When a component which is subscribed to an observable, gets destroyed by Angular, the subscription holds a reference to the component instance, preventing it from being available for garbage collection. Hence failing to unsubscribe an observable can lead to a memory leak.

The `unsubscribe()` method on the subscription object can be used to remove the subscription when it is no longer needed. Usually this is done within `ngOnDestroy`, which gets called during the destruction of the component.
 
{% highlight typescript %}
ngOnInit() {
  this.subscription = this.someService.getItems().subscribe(items => {
    this.items = items;
  });
}

ngOnDestroy() {
  this.subscription.unsubscribe();
}
{% endhighlight %}

# Manage multiple subscriptions

When there are multiple subscriptions code can get repetitive and harder to maintain.

{% highlight typescript %}
ngOnDestroy() {
  this.subscription1.unsubscribe();
  this.subscription2.unsubscribe();
  this.subscription3.unsubscribe();
}
{% endhighlight %}

As to avoid such repetition, these subscriptions may be attached to a parent `Subscription` via the `add` method. When the parent subscription is unsubscribed, its children are also unsubscribed.

{% highlight typescript %}
subscriptions = new Subscription();
ngOnInit() {
  this.subscriptions.add(subscription1);
  this.subscriptions.add(subscription2);
  this.subscriptions.add(subscription3);
}

ngOnDestroy() {
  this.subscriptions.unsubscribe();
}
{% endhighlight %}

# Use class inheritance

In order to keep the code clean, we can make an abstract class with a `subscriptions` property. Unsubscription can be handled at the abstract class level, reducing boilerplate from components.

{% highlight typescript %}
export abstract class BaseComponent implements OnDestroy {
    protected subscriptions = new Subscription();
    
    ngOnDestroy() {
        this.subscriptions.unsubscribe();
    }
}

export class SomeComponent extends BaseComponent {

    constructor(
        private someService: SomeService
    ) {
        super();
    }

    ngOnInit() {
      this.subscriptions.add(
        this.someService.getItems()
            .subscribe(items => {
                this.items = items;
             })
      );
    }
}

{% endhighlight %}

Any component requiring to subscribe can then extend the abstract class and just add any required subscriptions as a tear down using the `add` method.