---
layout: mypost
title: 友情链接
---

> 吾身如浮萍，不敢言再会，幸得天眷顾，得挚友二三

欢迎各位朋友与我建立友链，如需申请友链请到[留言板](chat.html)并仿照我的信息留言以下内容（头像不必须），一般两天之内就会添加上的，本站的友链信息如下

```
名称：{{ site.title }} 
描述：{{ site.description }}
地址：{{ site.domainUrl }}{{ site.baseurl }}
头像：{{ site.domainUrl }}{{ site.baseurl }}static/img/logo.jpg
```

<ul>
  {%- for link in site.links %}
  <li>
    <p><a href="{{ link.url }}" title="{{ link.desc }}" target="_blank" >{{ link.title }}</a></p>
  </li>
  {%- endfor %}
</ul>
