---
title: "May Update I"
description: ""
date: 2022-05-06T18:12:10+01:00
lastmod: 2022-05-06T18:12:10+01:00
draft: false
images: []
---

- Query API: redefined role
    - The dashboard (implemented with Grafana) was supposed to periodically query an API which provides metrics and status about the platform, which is the Query API. Throughout development, the use of Prometheus, along with Pushgateway for a push-based approach to metrics reporting (periodic pooling), was considered as it integrated well with Grafana and Hudi itself provides native support for metrics reporting to Prometheus (through the mentioned Pushgateway server). While the team was focused on integrating Prometheus, the role of the Query API in this infrastructure began to be questioned, until clarification with the supervisor cemented that, while it would have been more desirable to have the Query API developed standalone from the dashboard, for a functioning product an approach solely based on Prometheus would work. If time allows it, this issue could be revisited and the backend could be made more personalized with a custom Query API
