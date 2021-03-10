---
title: SQL Server page split
excerpt_separator: "<!--more-->"
modified: 2021-03-09
permalink: /page-split
categories:
  - Blog
  - SQL
  - SQL Server
  - internals
tags:
  - Post Formats
  - readability
  - standard
---

![visitors](https://visitor-badge.glitch.me/badge?page_id=includewareok.blog.2021-03-09-page-split")

## Introducción
Un `page split` ocurre cuando no hay más espacio libre en una página de datos para soportar las operaciones de `INSERTS` o `UPDATES`, por esa razón SQL Server envia parte del contenido de dicha página a otra. En esta entrada vamos a ver como ocurre dicha operación y el impacto que tiene a nivel de performance en nuestro trabajo diario. 
<!--more-->

**:information_source:** 
Esto es un WIP y se actualizará pronto.
{: .notice}