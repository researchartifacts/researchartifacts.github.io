---
title: "Repository Statistics"
permalink: /repo_stats.html
top_repos_url: /assets/data/top_repos.json
yearly_chart_url: /assets/data/repo_stats_yearly.json
yearly_chart_area: all
---

GitHub stars and forks for artifact repositories across all tracked conferences.

{% if site.data.repo_stats.overall %}

| | |
|---|---|
| **GitHub Repos** | {{ site.data.repo_stats.overall.github_repos }} |
| **Total Stars** | {{ site.data.repo_stats.overall.total_stars }} |
| **Median Stars** | {{ site.data.repo_stats.overall.median_stars }} |
| **P25 Stars** | {{ site.data.repo_stats.overall.p25_stars }} |
| **P75 Stars** | {{ site.data.repo_stats.overall.p75_stars }} |
| **Max Stars** | {{ site.data.repo_stats.overall.max_stars }} |
| **Total Forks** | {{ site.data.repo_stats.overall.total_forks }} |
| **Median Forks** | {{ site.data.repo_stats.overall.median_forks }} |
| **P25 Forks** | {{ site.data.repo_stats.overall.p25_forks }} |
| **P75 Forks** | {{ site.data.repo_stats.overall.p75_forks }} |

{% endif %}

{% if site.data.repo_stats.by_conference.size > 0 %}

## By Area

| Area | GitHub Repos | Total Stars | Median Stars | P25 Stars | P75 Stars | Total Forks | Median Forks | P25 Forks | P75 Forks | Max Stars |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
{% for a in site.data.repo_stats.by_area %}| **[{{ a.name | capitalize }}]({{ '/' | append: a.name | append: '/repo_stats.html' | relative_url }})** | {{ a.github_repos }} | {{ a.total_stars }} | {{ a.median_stars }} | {{ a.p25_stars }} | {{ a.p75_stars }} | {{ a.total_forks }} | {{ a.median_forks }} | {{ a.p25_forks }} | {{ a.p75_forks }} | {{ a.max_stars }} |
{% endfor %}

## Top Repositories

{% include top_repos_table.html %}

## Median Stars & Forks by Year

{% include repo_yearly_chart.html %}

---

**Data:** [All Conferences]({{ '/assets/data/top_repos.json' | relative_url }}) | [Systems]({{ '/assets/data/systems_top_repos.json' | relative_url }}) | [Security]({{ '/assets/data/security_top_repos.json' | relative_url }}) | [Yearly Stats]({{ '/assets/data/repo_stats_yearly.json' | relative_url }})

{% else %}

*Repository statistics have not been collected yet. Run the pipeline with `generate_repo_stats.py` to populate this data.*

{% endif %}
