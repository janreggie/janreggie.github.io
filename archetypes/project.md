---
title: "{{ replace .Name "-" " " | title }}"
type: project
date: {{ .Date }}
publishdate: {{ now.Format "2006-01-02" }}
lastmod: {{ now.Format "2006-01-02" }}
draft: true
description: "Insert the description here"
tags:
    - "tag 1"
categories:
    - "project"
link: "https://example.com" # link to output, documentation, or Github page
---

Summary

<!--more-->

## Post Header

CONTENT
