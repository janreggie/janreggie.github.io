---
title: "{{ replace .Name "-" " " | title }}"
type: audio
date: {{ .Date }}
publishdate: {{ now.Format "2006-01-02" }}
lastmod: {{ now.Format "2006-01-02" }}
draft: true
description: "Text about this post"
tags:
  - "tag 1"
categories:
  - Music
spotify: "" # Spotify URI as spotify:track:6dMMdTgB1LcJVV3tDuET29
---

Summary

<!--more-->

## Post Header

CONTENT
