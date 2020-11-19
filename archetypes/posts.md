---
title: {{ replaceRE "[0-9]{2,}" "" .Name | replaceRE "^-*" "" | replaceRE "-" " " | title }}
date: {{ .Date }}
lastmod: 
author: Dariusz Parys

description: 
categories: []
tags: []

draft: true
enableDisqus : false
enableMathJax: false
disableToC: false
disableAutoCollapse: true
---