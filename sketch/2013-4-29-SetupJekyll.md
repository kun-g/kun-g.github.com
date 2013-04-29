---
layout: default
title: "设置Jekyll"
categories: Tools
---

![Git]({{site.img_url}}/bkg.png)


代码高亮:
* gist
* google-code-prettify
* pygments
* jsfiddle

{% highlight ruby linenos %}
def foo
  puts 'foo'
end
{% endhighlight %}

{% highlight c linenos %}
int inc(int arg) {
  return arg + 1;
}
{% endhighlight %}

<p>{{ page.content | number_of_words }}</p>
