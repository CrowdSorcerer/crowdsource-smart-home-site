---
title: "May Update II"
description: ""
date: 2022-05-02T18:12:10+01:00
lastmod: 2022-05-02T18:12:10+01:00
draft: false
images: []
---

- Added Epic: Ingest API optimizations (1 week)
  - The Ingest API, while functional, has some details that still have to be addressed: limiting the rate of requests for each host (the operations are costly and have no need of being done in short periods of time) and defining the compression used for the payload on data upload.