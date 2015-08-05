---
layout: post
title:  "RSpec stub_const"
date:   2015-07-20 21:11:09
author: "Jeffrey Jurgajtis"
---

RSpec (2.11+) provides `stub_const` for the purpose of stubbing constants. This
method is not new and chances are you've seen it before. However, recently I've
been paying more attention to stubbing dependencies that shouldn't have an
affect on the outcome of my tests. I'll give a quick example of how
`stub_const` has helped.

I was recently working on a Rails application that allowed users to read and 
write reviews for local businesses. I began on a task to prevent users from
editing reviews that were older than 90 days.

I started with a few tests to add an instance method to `Review` to determine
if a review was within the edit period.

{% highlight ruby %}
# review_spec.rb
describe "#within_edit_period?" do
  let(:today)               { Time.zone.local(2015, 7, 11) }
  let(:ninety_days_ago)     { Time.zone.local(2015, 4, 12) }
  let(:ninety_one_days_ago) { Time.zone.local(2015, 4, 11) }

  before { Timecop.freeze(today) }
  after  { Timecop.return }

  it "returns true when created within edit period" do
    review = build_stubbed(:review, created_at: ninety_days_ago)
    expect(review.within_edit_period?).to eq true
  end

  it "returns false when created outside edit period" do
    review = build_stubbed(:review, created_at: ninety_one_days_ago)
    expect(review.within_edit_period?).to eq false
  end
end

{% endhighlight %}

The tests passed by adding the following constant and instance method to
`Review`.

{% highlight ruby %}
# review.rb
class Review < ActiveRecord::Base
  EDIT_PERIOD_IN_DAYS = 90

  ...

  def within_edit_period?
    created_at.advance(days: EDIT_PERIOD_IN_DAYS) > Time.zone.today
  end
end

{% endhighlight %}

At this point, my tests were dependent on the value of `EDIT_PERIOD_IN_DAYS`.
If the client wanted to change the edit period to something else, such as 30
days, updating `EDIT_PERIOD_IN_DAYS` would cause the tests to fail. However,
`.within_edit_period?` would still work as expected.

I used `stub_const` to replace the original value of `EDIT_PERIOD_IN_DAYS` for
the duration of the tests. With this addition, no time will be spent updating
tests in the future should the edit period change for reviews.

{% highlight ruby %}
# review_spec.rb
describe "#within_edit_period?" do
  ...

  before {
    Timecop.freeze(today)
    stub_const("Review::EDIT_PERIOD_IN_DAYS", 90)
  }
  
  after  { Timecop.return }

  ...
end

{% endhighlight %}
