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
- JSON (recommended)
- XML
- CSV

The data returned is compressed in zip format.

JSON format is the recommended method because the Home Assistant data has a JSON-like structure, which is already properly embedded in the final JSON resource.

**Default link**: [https://smarthouse.av.it.pt/api/export](https://smarthouse.av.it.pt/api/export)

*Note: this documentation is also present as a Swagger documentation page on* [https://smarthouse.av.it.pt/api/export/ui](https://smarthouse.av.it.pt/api/export/ui)

## Endpoints

All endpoints are relative to `/api/export`.

| Endpoint | Method | Header parameters | Path parameters | Query parameters | Request body |
| --- | --- | --- | --- | --- | --- |
| /dataset | GET | - | - | `formats`, `date_from`, `date_to`, `types`, `units` | - |
| /formats | GET | - | - | - | - |

Parameter definitions:

- **formats** (*required*): the case insensitive strings representing the dataset's output formats, which can be one of the following supported formats:
  - JSON (recommended)
  - XML
  - CSV
- **date_from**: only data from this date forwards will be extracted (UTC+0, in ISO 8601 format), inclusive
- **date_to**: only data from this date backwards will be extracted (UTC+0, in ISO 8601 format), inclusive
- **types**: only data from these types of producer will be extracted (e.g. sensor). An array of types can be provided
- **units**: only data which is represented in the specified units of measurement will be extracted (e.g. GHz). An array of units can be provided

## Operations

### Data extraction

Extract the data into CKAN compliant formats. The available formats can be programatically obtained in a comma-separated string by calling the `/formats` endpoint.

The data is provided in a zip archive, which when extracted provides the following files:

- `crowdsorcerer_extract_{date}.{format}`: file containing the data extracted. `date` is the date when the file was requested, and `format` is the format of the output file requested. If many formats were specified, then there will be multiple data files with the different formats
- `crowdsorcerer_extract_{date}_metadata.json`: file containing the metadata, which can be supplied in a request to a CKAN server

The metadata is laid out as the description of a CKAN dataset. Below is the example metadata of a dataset extracted with two output formats.

```json
{
    "name": "crowdsorcerer-extract",
    "title": "CrowdSorcerer extract",
    "author": "CrowdSorcerer",
    "license_id": "cc-by-sa",
    "notes": "Crowdsourced smart home data collected from the CrowdSorcerer open source project. More info on https://smarthouse.av.it.pt",
    "resources": [
        {
            "package_id": "crowdsorcerer-extract",
            "url": "upload-json",
            "format": "json"
        },
        {
            "package_id": "crowdsorcerer-extract",
            "url": "upload-csv",
            "format": "csv"
        }
    ],
    "extras": [
        {
            "key": "date_from",
            "value": "2022-04-28"
        },
        {
            "key": "date_to",
            "value": "2022-06-03"
        },
        {
            "key": "types",
            "value": "['sensor']"
        },
        {
            "key": "units",
            "value": "any"
        }
    ]
}
```

The license is [CC BY-SA](https://creativecommons.org/licenses/by-sa/4.0/).

The `extras` key contains the list of effective filters applied on the extraction of this dataset. The filters, if not specified, become:
- `date_from`: the date of the first insertion to the data lake
- `date_to`: the most recent date allowed to be extracted from the data lake (yesterday)
- `types` and `units`: "any"

## Errors

Some errors may be encountered, which may be the fault of the client or a problem with the server. Below are the custom errors for this API:

- **Bad date format (400)**: the supplied date parameter (either `date_from` or `date_to`) has an incorrect format, which should be ISO 8601 (`yyyy-mm-dd`). The considered timezone is UTC+0, so there may be cases where this consideration may make a difference, but it shouldn't be the cause of this error.
- **Unsupported exportation format (400)**: the supplied exportation format is not supported. The format string should be the same as the supported formats listed above, without case sensitivity. The API usually provides feedback on the unsupported string that was presented on the request and the strings that are actually accepted.
- **Empty dataset (204)**: the provided query filters produce a dataset without data columns or rows. The filtering process produced a dataset without any data about the homes. This can happen if a query filter restricts the data with constraints that no rows comply with, such as a `units` filter specifying a single unit of measurement that doesn't exist in the data lake.
- **Feature space is too large (500)**: the filtered dataset has too many columns to efficiently work with it. Consider applying more restrictive filters, and obtain a large dataset in parts. This happens because a large number of columns makes it expensive to convert the data to the final dataset on the API, and so the request has to be terminated.
