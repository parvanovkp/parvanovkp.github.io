---
layout: page
title: Projects
permalink: /projects/
description: This is a collection of most of the projects I have worked on so far in my career.
nav: true
nav_order: 2
display_categories: [School, Personal]
horizontal: false
published: true
---

<!-- pages/projects.md -->
<div class="projects">
  {%- if site.enable_project_categories and page.display_categories %}
  <!-- Display categorized projects -->
  <div class="container">
    <div class="row">
      {%- for category in page.display_categories %}
      <div class="col-md-6">
        <h2 class="category">{{ category }}</h2>
        {%- assign categorized_projects = site.projects | where: "category", category -%}
        {%- assign sorted_projects = categorized_projects | sort: "importance" %}
        <!-- Generate cards for each project -->
        {% if page.horizontal -%}
        <div class="container">
          <div class="row row-cols-1 row-cols-md-2 g-4">
            {%- for project in sorted_projects -%}
            <div class="col">
              {% include projects_horizontal.html %}
            </div>
            {%- endfor %}
          </div>
        </div>
        {%- else -%}
        <div class="grid">
          {%- for project in sorted_projects -%}
            {% include projects.liquid %}
          {%- endfor %}
        </div>
        {%- endif -%}
      </div>
      {% endfor %}
    </div>
  </div>
  {%- else -%}
  <!-- Display projects without categories -->
  {%- assign sorted_projects = site.projects | sort: "importance" -%}
  <!-- Generate cards for each project -->
  {% if page.horizontal -%}
  <div class="container">
    <div class="row row-cols-1 row-cols-md-2 g-4">
      {%- for project in sorted_projects -%}
      <div class="col">
        {% include projects_horizontal.html %}
      </div>
      {%- endfor %}
    </div>
  </div>
  {%- else -%}
  <div class="grid">
    {%- for project in sorted_projects -%}
      {% include projects.liquid %}
    {%- endfor %}
  </div>
  {%- endif -%}
  {%- endif -%}
</div>