---
title: "Example Project: Milton, MA Community Maps"
date: 2023-02-18T07:20:32-05:00
draft: false
weight: 3
tags: ["geospatial", "community"]
categories: ["portfolio"]
resources:
- name: "featured-image"
  src: "milton_map.png"
---

I recently did an [exploratory analysis](https://miltonmaps.hashadatascience.com) combining several publicly
available geospatial datasets about the town of Milton, MA (where I live!) and surrounding communities.  I had a few
goals in putting this together:

1. Provide an example of a Python project structure that is portable, transparent and reproducibile using
   modern tools for data versioning, document generation, and python environment management.
   Namely, [dvc](https://dvc.org/) for data version control and DAGs, [sphinx](https://www.sphinx-doc.org/en/master)
   for document generation, and [conda](https://conda.io/) for environment management.
2. Build awareness in the community of Milton civic organizations of available sources of map data and
   how they might be used to answer questions of local interest.  Provide a starting foundation for
   anyone in town who may want to work with this data in Python.
3. Learn more about geospatial analytics tools in Python!

Checkout the project [here](https://miltonmaps.hashadatascience.com/)!