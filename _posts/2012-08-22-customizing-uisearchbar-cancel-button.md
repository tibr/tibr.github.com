---
layout: post
title: Customizing UISearchBar Cancel Button Appearance
---
Just a quick tip: If you want to customize the cancel button of a `UISearchBar` you can do that by using the UIAppearance protocol of `UIBarButtonItem` like this:

{% highlight objc %}
[[UIBarButtonItem appearanceWhenContainedIn:[UISearchBar class], nil]
  setBackgroundImage:myBackgroundImage
  forState:UIControlStateNormal
  barMetrics:UIBarMetricsDefault];
{% endhighlight %}

You could also customize the appearance of every `UIButton` contained in a `UISearchBar` but that would also apply to the clear button inside of the `UISearchBarTextField` so I think you're better off customizing the `UIBarButtonItem`.