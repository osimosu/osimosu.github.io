---
layout: "post"
title:  "Separating jhipster back-end and front-end into two projects?"
date:   "2022-10-18 15:39:00 -0500"
categories: jhipster, entity, docker
---

This question was asked on [Stackoverflow](https://stackoverflow.com/questions/29028391/separating-jhipster-back-end-and-front-end-into-two-projects){:target="_blank"}.

- Generate the backend application 

```bash
jhipster --skip-client -db=sql --auth=jwt --search-engine=elasticsearch
```

- Generate the frontend application

```bash
jhipster --skip-server
```

- Generate entities for backend using your jdl file

```bash
jhipster import-jdl myapp.jdl --interactive
```
- Generate entities for frontend using same jdl file

```bash
jhipster import-jdl myapp.jdl --interactive
```