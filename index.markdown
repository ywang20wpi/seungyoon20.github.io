---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
title: Index
---
<div class="tags-expo">
  <div class="tags-expo-list">
    {% for tag in site.categories %}
    <a href="#{{ tag[0] | slugify }}" class="post-tag">{{ tag[0] }}</a>
    {% endfor %}
  </div>
  <hr/>
  <div class="tags-expo-section">
    {% for tag in site.categories %}
    <h3 id="{{ tag[0] | slugify }}">{{ tag[0] }}</h3>
    <ul class="post-list">
      {% for post in tag[1] %}
      <li>
        <!-- <span class="post-meta">{{ post.date | date_to_string }}</span> -->
        <a class="post-link" href="{{ site.baseurl }}{{ post.url }}">
        {{ post.title }}
      
      </a>
    </li>
      {% endfor %}
    </ul>
    {% endfor %}
  </div>
</div>