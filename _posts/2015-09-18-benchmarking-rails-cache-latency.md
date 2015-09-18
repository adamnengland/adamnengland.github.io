---
layout: post
title: Rails Benchmarking - Cache Store Latency
date: 2015-09-18 07:46:00.000000000 -06:00
categories:
- rails
tags:
- rails
- memcached
status: publish
type: post
published: true
author: Adam N England
---

If you've ever built a sizeable Rails app before, you know that efficient cache use can really supercharge your application. The [storEDGE](http://www.storedge.com) platform has always tried to offer the best performance to our users by keeping our memcached processes on the same instances as our application servers.

While this is fast, there are downsides. Adding new servers takes more effort, since memcached has to installed on every box. We also lose the ability to manage and scale our cache independently of our application servers. Finally, application servers may need to be tuned for CPU performance, while RAM is more likely the constraint on cache servers. Given all of this, it is often a good idea to put your cache on dedicated hardware.

The real question is, how much will network latency hurt us?  Let's try a benchmark to find out. The following code simulates a read-heavy cache load in a rails application. You could run this through a rails console.  In this case, I put it in a rake task for convenience. You can see the sample code [here](https://github.com/adamnengland/rails-cache-latency-benchmark)

{% highlight ruby linenos %}
cacheval = [(0..1000).map{ rand(36).to_s(36) }.join]
Benchmark.bm { |x|
  x.report("caching:") {
    Rails.cache.write("test", cacheval)
    1000.times do
      if !Rails.cache.exist?("test")
        raise
      else
        Rails.cache.fetch("test")
      end
    end
    Rails.cache.delete("test")
  }
}
{% endhighlight %}

For the first round, I'm running on my 2014-era MacBook Pro.

### 1000 iterations, developer environment

#### Local Memcache
{% highlight bash %}
$ rake cache_benchmark:test
       user     system      total        real
caching:  0.160000   0.050000   0.210000 (  0.244702)
{% endhighlight %}

Pretty fast, lets try a remote memcached service. [Memcachier](http://www.memcachier.com) is a popular option.

#### Remote Memcahier Instance
{% highlight bash %}
$ rake cache_benchmark:test
       user     system      total        real
caching:  0.560000   0.190000   0.750000 (114.471950)
{% endhighlight %}

Pretty dramatic, huh? Based on this, you'd have to be pretty crazy to run memcached anywhere except ON the app server. However, we've got a pretty major flaw here: I'm running these benchmarks from my home in Kansas City, MO. The memcachier server is running on Amazon AWS in the us-east region. What happens if I run these benchmarks on an ec2 box in us-east?

### 1000 iterations, running on ec2 us-east m3.medium  

#### Local Memcache
{% highlight bash %}
$ rake cache_benchmark:test
       user     system      total        real
caching:  0.320000   0.030000   0.350000 (  0.419922)
{% endhighlight %}

#### Remote Memcachier instance
{% highlight bash %}
$ rake cache_benchmark:test
       user     system      total        real
caching:  0.220000   0.040000   0.260000 (  2.846208)
{% endhighlight %}

Now, that is more like it. When we simulate the network latency of a production environment, with cache servers in close proximity to the application server, we get some much more reasonable results.

If we are dealing with AWS, we may also want to consider [Amazon Elasticache](https://aws.amazon.com/elasticache/). After spinning up an Elasticache instance, and pointing our benchmarks at that, we get the following:

#### Elasticache Memcache - cache.t2.micro instance on us-east, same VPC
{% highlight bash %}
$ rake cache_benchmark:test
       user     system      total        real
caching:  0.240000   0.030000   0.270000 (  3.874526)
{% endhighlight %}

Surprisingly, Elasticache is a little bit slower than Memcachier in this example. In a production application, the difference is probably negligible, however, I expected Elasticache to perform better.

At this point, we know that using a remote server does suffer from a good bit of latency compared to a local memcached process. However, we don't really see much difference between Elasticache and a service like Memcachier, provided that your cache is in the same region. Let's try a bonus caching option - Redis. I'm a fan of [Compose.io](http://www.compose.io) for simple, scalable databases. While Compose does not offer memcached servers, we can try a redis cache?

#### Compose.io - 256 MB Redis Cluster in us-east
{% highlight bash %}
$ rake cache_benchmark:test
       user     system      total        real
caching:  0.230000   0.030000   0.260000 (  1.862227)
{% endhighlight %}

Compose looks to be faster than either of the memcache options. Lets kick it up to 100,000 times, and see how the numbers play out.

### 100,000 iterations, running on ec2 us-east m3.medium  

#### Local Memcache
{% highlight bash %}
$ rake cache_benchmark:test
       user     system      total        real
caching: 27.620000   6.110000  33.730000 ( 42.674231)
{% endhighlight %}

#### Local Redis
{% highlight bash %}
$ rake cache_benchmark:test
       user     system      total        real
caching: 26.070000   5.100000  31.170000 ( 41.643664)
{% endhighlight %}

#### Redis on Compose.io
{% highlight bash %}
$ rake cache_benchmark:test
       user     system      total        real
caching: 21.000000   3.750000  24.750000 (200.054559)
{% endhighlight %}

#### Elasticache
{% highlight bash %}
$ rake cache_benchmark:test
       user     system      total        real
caching: 21.770000   4.630000  26.400000 (387.568663)
{% endhighlight %}

### Summary Data

If performance is your only concern, then keep your cache on your application server. However, if the management and scalability benefits of putting your cache on dedicated servers appeal to you, you have a lot of good options. Run the benchmarks in your own environment to make the right decision, since your hosting location options will probably be the single largest factor in keeping your cache times fast.

<table>
    <tr>
      <th>Configuration</th>
      <th>Time (100,000 iterations in seconds)</th>
      <th>Comparison to Baseline</th>
    <tr>
        <td>Redis on Localhost</td>
        <td>41.643664</td>
        <td> -- </td>
    </tr>
    <tr>
        <td>Memcached on Localhost</td>
        <td>42.674231</td>
        <td> 102% </td>
    </tr>
    <tr>
        <td>Redis on Compose.io</td>
        <td>200.054559</td>
        <td> 480% </td>
    </tr>
    <tr>
        <td>Memcached on Elasticache</td>
        <td>387.568663</td>
        <td> 930% </td>
    </tr>
</table>
