---
layout: post
title:  "Using Service Objects to Refactor Models"
date:   2015-08-19 21:11:09
author: "Jeffrey Jurgajtis"
---

It's been at least a few years since I've heard the "fat models, skinny
controllers" design style get any attention in the Rails community. This is
great because that pattern has led (me to write) to some very unmaintainable
code.

Since then, I've been exploring solutions to keep behavior, presentation, and
anything else not related to modeling objects out of models. In Rails terms, a
model is a class that inherits from `ActiveRecord::Base`. The responsibility of
these classes are to provide structure for the data in a system. One way that
I've been working to keep models inline with the [Single Responsiblity
Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle)
(SRP) is through the use of service objects.

### Sevice Object Definition

A service object encapsulates business logic and is responsible for performing
a single process in the application.

As an example of a process, lets say I had an application that integrated with
a payment processing service for subscription management. In the process of
canceling a subscription, two things would need to happen:

* 1). An API request to the payment processor to cancel the subscription.
* 2). An update to the local subscription record to set the `canceled_at` date.

Following the "fat model" pattern, this code would end up in
`models/subscription.rb` and result in something like this:

{% highlight ruby %}
class Subscription < ActiveRecord::Base
  belongs_to :user

  ...

  def cancel
    PaymentGateway.cancel_subscription!(
      subscription_id: payment_gateway_id,
      customer_id: user.payment_gateway_id
    )
    update(canceled_at: Time.zone.now)
  end
end
{% endhighlight %}

Knowing how to cancel a subscription in the payment processing service falls
outside the scope of a subscription's purpose. Moving this code to a service
will result in a clean and organized solution.

### Service Object Charateristics

Before moving the subscription cancellation code to a service object, I'll cover
a few guidelines that I've found work well for building and maintaining service
objects.

**One responsibility per service.** As mentioned earlier, service objects
encapsulate a _single_ process. This goes along with maintaining SRP and the
reason we're moving code to a service object in the first place. I've found
that making a service responsible for multiple processes usually results in
conditionals or `nil` values, which in turn results in less maintainable code.

**Instance methods, not class methods.** I try to avoid class methods in general
as I find they are less susceptible to change and lead to non object-oriented
results. Building a service as an object allows for code to be extracted to
methods, implementation details to be hidden via private methods, and objects
passed in to be accessible via instance variables.

### Subscription Cancellation Service

I'll name the service class `CancelSubscription` and add it to the
`app/services` directory.

In the original subscription model `cancel` method, the subscription's user
association (`belongs_to :user`) was used to get the user's
`payment_gateway_id`. If the subscription and user associations ever change, I
want the subscription cancellation service to be unaffected, so I'll pass in the
user. In addition, there will be no need for the subscription and user to be
publicly accessible, so I'll make the getters private.

{% highlight ruby %}
class CancelSubscription
  def initialize(subscription:, user:)
    @subscription = subscription
    @user         = user
  ene

  def process
  end

  private

  attr_reader :subscription, :user
end
{% endhighlight %}

Result objects provide a flexible way for representing the outcome of a service
object process. For example, if an exception is raised by the `PaymentGateway`
API adapter, I'll preserve the error via the result object.

Here's the completed service:

{% highlight ruby %}
class CancelSubscription
  Result = Struct.new(:success?, :error_message)

  def initialize(subscription:, user:)
    @subscription = subscription
    @user         = user
  end

  def process
    payment_gateway_cancel!
    subscription.update!(canceled_at: Time.zone.now)
    Result.new(true)
  rescue => e
    Result.new(false, e.message)
  end

  private

  def payment_gateway_cancel!
    PaymentGateway.cancel_subscription!(
      subscription_id: subscription.payment_gateway_id,
      customer_id: user.payment_gateway_id
    )
  end

  attr_reader :subscription, :user
end
{% endhighlight %}

Here is a look at the `CancelSubscription` being used in a controller:

{% highlight ruby %}
# controllers/subscription_cancellations_controller.rb

  def create
    cancellation_result = CancelSubscription.new(
      subscription: @subscription,
      user: current_user
    ).process

    if cancellation_result.success?
      redirect_to root_url
    else
      flash[:error] = result.error_message
      render :new
    end
  end
{% endhighlight %}

### Conclusion

I've found that service objects are a great alternative to cluttering models
with business logic and provide a meaningful way of representing the important
processes of an application.
