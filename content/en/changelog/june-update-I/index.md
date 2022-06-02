---
title: "June Update I"
description: ""
date: 2022-06-02T14:12:10+01:00
lastmod: 2022-06-02T14:12:10+01:00
draft: false
images: []
---

- The Ingest API is extremelly heavy, since for each upload operation the data is inserted with Hudi, which takes a lot of time. In order to prevent blocking API requests that take too long on the Home Assistant Aggregator and to not overload the data lake, the data in each request will be saved in memory and only inserted in predefined time intervals.
- Added Redis for increasing the performance of the Ingest API and caching Export API datasets. At first the uWSGI caching utility posed itself as a possible solution, but the very limited space for storing values (64KiB) when compared with Redis (512MiB) was a deterrent to both use cases.
