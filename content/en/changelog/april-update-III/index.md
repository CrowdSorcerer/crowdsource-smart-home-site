---
title: "April Update III"
description: ""
date: 2022-04-20T16:05:20+01:00
lastmod: 2022-04-20T16:05:20+01:00
draft: false
images: []
---

- Data Lake: downgraded PySpark version to 3.1.2 (from 3.2.1)
    - We found problems with this version when deleting data with Hudi (DateFormatter class  exception thrown by Py4J).