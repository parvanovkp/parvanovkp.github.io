---
layout: page
title: Projects
permalink: /projects/
description: This is a collection of most of the projects I have worked on so far in my career.
nav: true
nav_order: 2
display_categories: [Personal, School]
horizontal: false
---

<!-- pages/projects.md -->
<div class="projects">
  {%- if site.enable_project_categories and page.display_categories %}
    <!-- Display categorized projects -->
    {% for category in page.display_categories %}
      <a id="{{ category }}" href=".#{{ category }}">
        <h2 class="category">{{ category }}</h2>
      </a>
      {%- assign categorized_projects = site.projects | where: "category", category -%}
      {%- assign sorted_projects = categorized_projects | sort: "importance" %}
      <!-- Generate cards for each project -->
      {% if page.horizontal -%}
        <div class="container">
          <div class="row row-cols-1 row-cols-md-2 g-3">
          {%- for project in sorted_projects -%}
            <div class="col">
              {% include projects_horizontal.liquid %}
            </div>
          {%- endfor %}
          </div>
        </div>
      {%- else -%}
        <div class="row row-cols-1 row-cols-sm-2 row-cols-lg-3 g-3">
          {%- for project in sorted_projects -%}
            <div class="col">
              {% include projects.liquid %}
            </div>
          {%- endfor %}
        </div>
      {%- endif -%}
    {% endfor %}
  {%- else -%}
    <!-- Display projects without categories -->
    {%- assign sorted_projects = site.projects | sort: "importance" -%}
    <!-- Generate cards for each project -->
    {% if page.horizontal -%}
      <div class="container">
        <div class="row row-cols-1 row-cols-md-2 g-3">
        {%- for project in sorted_projects -%}
          <div class="col">
            {% include projects_horizontal.liquid %}
          </div>
        {%- endfor %}
        </div>
      </div>
    {%- else -%}
      <div class="row row-cols-1 row-cols-sm-2 row-cols-lg-3 g-3">
        {%- for project in sorted_projects -%}
          <div class="col">
            {% include projects.liquid %}
          </div>
        {%- endfor %}
      </div>
    {%- endif -%}
  {%- endif -%}
</div>

<style>
  .projects .category {
    margin-top: 2rem;
    margin-bottom: 1rem;
  }
  .projects .card {
    height: 100%;
  }
  .projects .card-img-top {
    height: 200px;
    object-fit: cover;
  }
  .projects .card-body {
    display: flex;
    flex-direction: column;
  }
  .projects .card-title {
    font-size: 1.1rem;
    margin-bottom: 0.5rem;
  }
  .projects .card-text {
    flex-grow: 1;
    font-size: 0.9rem;
  }
</style>
