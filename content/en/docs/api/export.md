---
title: "Export API"
description: "Export API for data dumping from the data lake into CKAN compliant formats."
lead: "Export API for data dumping from the data lake into CKAN compliant formats."
date: 2022-05-11T17:36:41+01:00
lastmod: 2022-05-11T17:36:41+01:00
draft: false
images: []
menu:
  docs:
    parent: "api"
weight: 600
toc: true
---

From this API anyone can extract the collected data into a CKAN compliant dataset in the following supported formats:
- XML
- JSON
- CSV

The data returned is compressed in zip format.

**Default link**: [https://smarthouse.av.it.pt/api/export](https://smarthouse.av.it.pt/api/export)

*Note: this documentation is also present as a Swagger documentation page on* [https://smarthouse.av.it.pt/api/export/ui](https://smarthouse.av.it.pt/api/export/ui)

## Endpoints

All endpoints are relative to `/api/export`.

| Endpoint | Method | Header parameters | Path parameters | Query parameters | Request body |
| --- | --- | --- | --- | --- | --- |
| /{format} | GET | - | `format` | `date_from`, `date_to`, `types`, `units` | - |

Parameter definitions:

- **format**: the case insensitive string representing the dataset's output format, which can be one of the following supported formats:
  - XML
  - JSON
  - CSV
- **date_from**: only data from this date forwards will be extracted (UTC+0, in ISO 8601 format), inclusive
- **date_to**: only data from this date backwards will be extracted (UTC+0, in ISO 8601 format), inclusive
- **types**: only data from these types of producer will be extracted (e.g. sensor). An array of types can be provided
- **units**: only data which is represented in the specified units of measurement will be extracted (e.g. GHz). An array of units can be provided

## Operations

### Data extraction

Extract the data into CKAN compliant formats. The data is provided in a zip archive, which when extracted provides the data in the specified file format.

At this moment, the datasets don't have any metadata.

## Errors

Some errors may be encountered, which may be the fault of the client or a problem with the server. Below are the custom errors for this API:

- **Bad date format (400)**: the supplied date parameter (either `date_from` or `date_to`) has an incorrect format, which should be ISO 8601 (`yyyy-mm-dd`). The considered timezone is UTC+0, so there may be cases where this consideration may make a difference, but it shouldn't be the cause of this error.
- **Unsupported exportation format (400)**: the supplied exportation format is not supported. The format string should be the same as the supported formats listed above, without case sensitivity. The API usually provides feedback on the unsupported string that was presented on the request and the strings that are actually accepted.
- **Empty dataset (204)**: the provided query filters produce a dataset without data columns or rows. The filtering process produced a dataset without any data about the homes. This can happen if a query filter restricts the data with constraints that no rows comply with, such as a `units` filter specifying a single unit of measurement that doesn't exist in the data lake.
