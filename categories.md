---
title: Categories
layout: page
permalink: "/category/"
main_nav: true
---
<div class="container">
  <div class="row justify-content-md-center">
  {% for category in site.categories %}
    {% capture cat %}{{ category | first }}{% endcapture %}
    <div class="col-md-3 col-lg-3 col-sm-6 category-div">
      <div>
        <strong>
          <a href="{{ site.baseurl }}/category/{{ cat | downcase }}">{{ cat }}</a>
        </strong>
      </div>
    </div>
    <!-- {% if forloop.last == false %}<hr>{% endif %} -->
  {% endfor %}
  </div>
</div>
<br>
