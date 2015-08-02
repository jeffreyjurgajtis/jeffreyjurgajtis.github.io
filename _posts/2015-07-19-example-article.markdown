---
layout: post
title:  "Example Article"
date:   2015-07-19 21:11:09
author: "Jeffrey Jurgajtis"
categories: jekyll update
---

Lorem ipsum dolor sit amet, et inermis facilisis quaerendum nec. Novum offendit ne est. Usu nemore explicari an, eam ne mazim dignissim, aeque appareat tacimates no his. No probo nullam cum, ad eos possit urbanitas adversarium. Posse nobis delicatissimi ut sit, est et aeque molestie.

Eu eos ignota tibique percipit, te sea doming perfecto perpetua, in dolor saepe legendos nec. At alterum molestie vel, odio ancillae ex usu. Ei propriae temporibus vix, ne forensibus ullamcorper qui. Omnesque deseruisse intellegam an quo, id quot mollis reprehendunt nam. Ei nec brute ceteros.

Eu eos ignota tibique percipit, te sea doming perfecto perpetua, in dolor saepe legendos nec. At alterum molestie vel, odio ancillae ex usu. Ei propriae temporibus vix, ne forensibus ullamcorper qui. Omnesque deseruisse intellegam an quo, id quot mollis reprehendunt nam. Ei nec brute ceteros.

{% highlight ruby %}
def save!
  return false if @account_form.valid?

  @stripe_account = save_stripe_account!
  @account = save_account!
rescue Stripe::InvalidRequestError => e
  @account_form.errors.add(:base, e.message)
  false
end

{% endhighlight %}

Lorem ipsum dolor sit amet, et inermis facilisis quaerendum nec. Novum offendit ne est. Usu nemore explicari an, eam ne mazim dignissim, aeque appareat tacimates no his. No probo nullam cum, ad eos possit urbanitas adversarium. Posse nobis delicatissimi ut sit, est et aeque molestie.

Eu eos ignota tibique percipit, te sea doming perfecto perpetua, in dolor saepe legendos nec. At alterum molestie vel, odio ancillae ex usu. Ei propriae temporibus vix, ne forensibus ullamcorper qui. Omnesque deseruisse intellegam an quo, id quot mollis reprehendunt nam. Ei nec brute ceteros.
