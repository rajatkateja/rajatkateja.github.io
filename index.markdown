---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---
<style>
ul.no-bullets {
  list-style-type: none; /* Remove bullets */
  padding: 0; /* Remove padding */
  margin: 0; /* Remove margins */
}
</style>
<ul class="no-bullets">
  {% for post in site.posts %}
    <li>
      [{{ post.date | date: "%Y-%m-%d" }}] <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
