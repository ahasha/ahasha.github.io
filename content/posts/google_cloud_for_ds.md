---
title: "Google Cloud for Data Science"
date: 2023-03-11T13:57:12-05:00
draft: true
---

Setting up Google Cloud for some light data science.

These are the steps I took running Mac OSX Ventura 13.1.


Download the appropriate google cloud CLI software from [google's website](https://cloud.google.com/sdk/docs/install)

```
$ gcloud init
```

To get python google cloud packages to work, you need to run the following command once.
```zsh
$ gcloud auth application-default login   
```