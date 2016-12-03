---
layout: master
title: Derek's Blog
---

<div class='home_box' id='home_left'>
  <h2><a href='http://feeds.feedburner.com/derek0883n' class='float-right'><img src='/images/subscribe-icon.gif' alt='Subscribe'/></a> Latest Blog Post (<a href='/'>More</a>)</h2>
  
<table class='post-list'>
{% for post in site.posts %}
    <tr>
      <th><a href='{{ post.url }}'>{{ post.title }}</a></th>
      <td>{{ post.categories }}</td>
      <td><a href='{{post.url}}#disqus_thread'>Comments</a></td>
    </tr>
{% endfor %}
</table>
</div>
