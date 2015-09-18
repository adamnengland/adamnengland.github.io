---
layout: post
title: Rapid Lo-Dash is now available!
date: 2014-11-26 07:46:00.000000000 -06:00
categories:
- nodejs
- Web Development
tags:
- javascript
- nodejs
status: publish
type: post
published: true
author: Adam N England
---
I'm pleased to announce that my newest video series, Rapid Lo-Dash is available for purchase through Packt Publishing.

You can get all the details and view the free sample here: [Rapid Lo-Dash](https://www.packtpub.com/web-development/rapid-lo-dash-video)

If you're a JavaScript developer, and you aren't using [Lo-Dash](https://lodash.com/) yet, you really need to check it out. It'll help you write some cleaner, faster, more maintainable code. Don't believe me?  Take a look at this example, and decide which code you would rather maintain.

{% highlight javascript %}
var numbers = [1, 2, 3, 4, 5, 6];
var odd_numbers = [];
for (var i = 0; i < numbers.length; i++) {
  if (numbers[i] % 2 == 1) {
    odd_numbers.push(numbers[i]);
  }
}
{% endhighlight %}

{% highlight javascript %}
var numbers = [1, 2, 3, 4, 5, 6];
var odd_numbers = _.filter(numbers, function(num) {
	return num % 2 == 1;
});
{% endhighlight %}

Rapid Lo-Dash walks you through the basics of setting up your development environment, and using Lo-Dash to work with arrays. We'll see examples of how to use Lo-Dash to work with object, collections, chaining, and get into some basic functional programming concepts.

[<img class="alignnone size-medium wp-image-446" src="/assets/5-clean-up-your-code-with-lo-dash-chains.png" alt="5.Clean Up Your Code with Lo-Dash Chains" style="width:80%;" />](/assets/5-clean-up-your-code-with-lo-dash-chains.png)

Thanks to the team at Packt Publishing for working with me to create this video series - hope you enjoy it.
