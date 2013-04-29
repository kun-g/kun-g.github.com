---
layout: post
title: "设置Jekyll"
tags: Tool Jekyll 
categories: Blog
cover: "http://twitter.github.io/bootstrap/assets/img/examples/slide-01.jpg" 
headLine: "我使用Jekyll的流程"
---

![Git]({{site.img_url}}/bkg.png)

  {% for p in site.related_posts %}
    <li>{{ p.date | date_to_string }} <a href="{{ p.url }}"> {{ p.title }}</a></li>
  {% endfor %}

调用json  
{% highlight js linenos %}
<script type="text/javascript">
$(document).ready(function() {
    $.getJSON("/cn/recent.json",
      function(data) {
      var content = "<ul class=\"compact recent\">";
      $.each(data,
        function(i, item) {
        content += "<li><span class=\"date\">" + item.date + "</span><a href=\"" + item.url + "\">" + item.title + "</a></li>";
        });

      content += "</ul>";
      $("#blog1-posts-list").append(content);
      });
    });
</script>
{% endhighlight %}


代码高亮:
* gist
* google-code-prettify
* pygments
* jsfiddle

{% highlight js linenos %}
var a = 0;
a += 1;
{% endhighlight %}

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
