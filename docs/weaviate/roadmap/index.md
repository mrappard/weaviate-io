---
title: Roadmap
sidebar_position: 0
# layout: layout-documentation
# solution: weaviate
# sub-menu: Roadmap
# title: Roadmap
# description: Weaviate roadmap
# tags: ['roadmap']
# sidebar_position: 0
# open-graph-type: article
# toc: false
---
import Badges from '/_includes/badges.mdx';

<Badges/>

:::caution UNDER CONSTRUCTION
This page is being migrated.
:::

## Overview

The following is an overview of features planned for Weaviate. By clicking the link, you can upvote the feature or engage in a discussion about it. You can also join our [Slack channel](https://join.slack.com/t/weaviate/shared_invite/zt-goaoifjr-o8FuVz9b1HLzhlUfyfddhw) to discuss the roadmap in more detail.

* The current version of Weaviate is **v||site.weaviate_version||**. You can check the version you're currently running at the [meta](/docs/weaviate/references/rest/meta.md) endpoint.
* Upvote an issue by clicking the 👍 emoji on the Github issue page

Please feel free to engage with us about the roadmap on [Weaviate's Github](https://github.com/semi-technologies/weaviate) or on [Slack](https://join.slack.com/t/weaviate/shared_invite/zt-goaoifjr-o8FuVz9b1HLzhlUfyfddhw).

## Features

<!-- ADDS PLANNED VERSIONS -->
<!-- {% for label in site.data.roadmap %}
{% if label[0] != 'backlog' %}
## {{ label[0] | replace: 'planned-', 'Planned for version ' | camelcase }}
{% assign description = label[1].description | strip_newlines %}
{% if description != '' %}
<small>{{ description }}</small>
{% endif %} -->

<!-- <ul class="list-group mb-4">
{% assign issues = label[1].items | sort: '+1' | reverse %}
{% for issue in issues %}
<li class="list-group-item">
    <a href="{{ issue.url }}" target="_blank">{{ issue.title }}</a> – 👍 {{ issue['+1'] }}
</li>
{% endfor %}
</ul>

{% endif %}
{% endfor %} -->

<!-- ADDS BACKLOG -->
## Backlog
<!-- <ul class="list-group mb-4">
{% assign backlog = site.data.roadmap['backlog'].items | sort: '+1' | reverse %}
{% for issue in backlog %}
<li class="list-group-item">
    <a href="{{ issue.url }}" target="_blank">{{ issue.title }}</a> – 👍 {{ issue['+1'] }}
</li>
{% endfor %}
</ul> -->

## More Resources

import DocsMoreResources from '/_includes/more-resources-docs.md';

<DocsMoreResources />