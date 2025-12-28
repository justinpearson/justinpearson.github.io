---
layout: base-layout
title: Tips
nav_name: tips
---

# Tips

Short tech notes and references.

<ul class="tips-list">
{% assign sorted_tips = site.tips | sort: 'date' | reverse %}
{% for tip in sorted_tips %}
<li>
    <a href="{{ tip.url }}">
        <div class="tip-list-title">{{ tip.title }}</div>
        <div class="tip-list-meta">
            <span class="tip-list-date">{{ tip.date | date: "%Y-%m-%d" }}</span>
            {% if tip.tags %}
            <span class="tip-list-tags">
                {% for tag in tip.tags %}
                <span class="tag">{{ tag }}</span>
                {% endfor %}
            </span>
            {% endif %}
        </div>
    </a>
</li>
{% endfor %}
</ul>
