---
layout: post
title:  "Unsubscribe from RxJS Observables"
---

![](/rmb/assets/rxjsng.png)

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

# Manage subscriptions via class inheritance

When there are multiple subscriptions code can get repetitive and harder to maintain.

{% highlight typescript %}
ngOnDestroy() {
  this.subscription1.unsubscribe();
  this.subscription2.unsubscribe();
  this.subscription3.unsubscribe();
}
{% endhighlight %}

In order to keep the code clean, we can make an abstract class with `subscriptions` declared as a `protected`. 

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