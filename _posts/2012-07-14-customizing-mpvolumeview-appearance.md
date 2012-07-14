---
layout: post
title: Customizing MPVolumeView Appearance
---
Everybody loves MPVolumeView, it's the best way to control the volume when playing audio or video on the iPhone. And everybody loves custom graphics. iOS 5 introduced the appearance API which allows us to customize the appearance of UISliders.

However, MPVolumeView doesn't offer these customization options because it doesn't expose its UISlider. So what do you do if your designer creates an awesome looking volume slider? Write your own MPVolumeView? That's probably a bad idea...

In this post, I'll show you how to customize the appearance MPVolumeView to go from this

![MPVolumeView](/img/MPVolumeView_default.png)

to this

![MPVolumeView styled](/img/MPVolumeView_styled.png)

### Accessing the UISlider

First, we need to find the UISlider in the MPVolueViews subviews. Luckily for us it only contains one UISlider so we can find it like this:

{% highlight objc %}
for (id current in volumeView.subviews){
	if ([current isKindOfClass:[UISlider class]]) {
		UISlider *volumeSlider = (UISlider *)current;
		// customization code
	}
}
{% endhighlight %}

### Customizing the slider

Now that we have access to the slider, we can customize it with the appearance API. To match the designed slider, we need to customize the minimum track image, the maximum track image and the thumb image, you can grab the images I used <a href="/assets/VolumeSliderAssets.zip" target="_blank">right here</a>.

{% highlight objc %}
UIImage *minimumTrackImage = [[UIImage imageNamed:@"VolumeSliderMinTrackImage"]
					  resizableImageWithCapInsets:UIEdgeInsetsMake(0, 6, 0, 6)];
[volumeSlider setMinimumTrackImage:minimumTrackImage forState:UIControlStateNormal];
UIImage *maximumTrackImage = [[UIImage imageNamed:@"VolumeSliderMaxTrackImage"]
					  resizableImageWithCapInsets:UIEdgeInsetsMake(0, 6, 0, 6)];
[volumeSlider setMaximumTrackImage:maximumTrackImage forState:UIControlStateNormal];
[volumeSlider setThumbImage:[UIImage imageNamed:@"VolumeSliderThumb"]
				   forState:UIControlStateNormal];
{% endhighlight %}

This gets us quite close, but we're not done yet:

![MPVolumeView slider styled](/img/MPVolumeView_appearance.png)

### Fixing thumb alignment

Obviously, the thumb alignment is wrong. This is because the UISlider used in the MPVolumeView acutally is an <a href="https://github.com/rpetrich/iphoneheaders/blob/master/MediaPlayer/MPVolumeSlider.h" target="_blank">MPVolumeSlider</a> which overrides thumbRectForBounds:trackRect:value: to move the thumb a few pixels down.

This is where the fun begins - to change the implementation of that method in the MPVolumeSlider we need to swizzle it. Create a UISlider subclass which simpy returns the default UISlider result:

{% highlight objc %}
- (CGRect)thumbRectForBounds:(CGRect)bounds
                   trackRect:(CGRect)rect
                       value:(float)value
{
    return [super thumbRectForBounds:bounds
                           trackRect:rect
                               value:value];
}
{% endhighlight %}

In our customization code, we exchange the implementation from MPVolumeSlider with the implementation from our newly created subclass (I named it SFYVolumeSlider)

{% highlight objc %}
Method originalThumbRect = class_getInstanceMethod(NSClassFromString(@"MPVolumeSlider"),
							@selector(thumbRectForBounds:trackRect:value:));
Method newThumbRect = class_getInstanceMethod([SFYVolumeSlider class],
					   @selector(thumbRectForBounds:trackRect:value:));
method_exchangeImplementations(originalThumbRect, newThumbRect);
{% endhighlight %}

When we run our app, we see that we reached our goal:

![MPVolumeView with fixed alignment](/img/MPVolumeView_alignment_fixed.png)

### Bonus: Changing the track size

Let's imagine that for some reason we want the track to be higher. To do that a UISlider subclass overrides trackRectForBounds:. The thing is, MPVolumeSlider doesn't override this method so we can't exchange implementations. After adding the method to our custom subclass that returns a track that's a few pixels higher

{% highlight objc %}
- (CGRect)trackRectForBounds:(CGRect)bounds
{
    CGRect calculatedRect = [super trackRectForBounds:bounds];
    calculatedRect.size.height = 18.0f;
    return calculatedRect;
}
{% endhighlight %}

we need to add it to the MPVolumeView:

{% highlight objc %}
class_addMethod(NSClassFromString(@"MPVolumeSlider"),
                @selector(trackRectForBounds:),
                class_getMethodImplementation([SFYVolumeSlider class],
                 @selector(trackRectForBounds:)),
                "{CGRect={CGPoint=dd}{CGSize=dd}}@:");
{% endhighlight %}

Looks crappy but you get the idea:

![MPVolumeView with increased track height](/img/MPVolumeView_changed_height.png)