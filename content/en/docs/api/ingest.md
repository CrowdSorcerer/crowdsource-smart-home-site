---
title: "Ingest API"
description: "Ingest API for data ingestion into the data lake."
lead: "Ingest API for data ingestion into the data lake."
date: 2022-05-11T17:36:41+01:00
lastmod: 2022-05-11T17:36:41+01:00
draft: false
images: []
menu:
  docs:
    parent: "api"
weight: 999
toc: true
---

This API should be used solely by the Home Assistant Aggregator component, but there are no restrictions on where the requests come from, since those details can easily be forged.

**Default link**: [https://smarthouse.av.it.pt/api/ingest](https://smarthouse.av.it.pt/api/ingest)

*Note: this documentation is also present as a Swagger documentation page on* [https://smarthouse.av.it.pt/api/ingest/ui](https://smarthouse.av.it.pt/api/ingest/ui)

## Endpoints

All endpoints are relative to `/api/ingest`.

| Endpoint | Method | Header parameters | Path parameters | Query parameters | Request body |
| --- | --- | --- | --- | --- | --- |
| /data | POST | Home-UUID (required) | - | - | Home data (binary) |
| /data | DELETE | Home-UUID (required) | - | - | - |

Parameter definitions:

- **Home-UUID**: the UUID of the home relative to the operation (add data from home with UUID or delete all data of home with UUID). It should be in the proper UUID format.

Request body definitions:

- **Home data**: encoded JSON string compressed with `zlib` (encoding is UTF-8). This contains data about the home with the specified UUID, to be stored in the data lake. It is assumed that the data provided is already anonymized by the Home Assistant Aggregator component.

## Operations

### Data upload

Data upload is performed through the POST method on `/data`.

Through the Home Assistant Aggregator, each home produces a data payload that is inserted into the platform as it is received.

Each of these payloads is associated with an UUID, which is stored within the Aggregator instances and is sent along with the data.
The data upload to the platform is done publicly, with no proper authentication.

*Note: as it stands, this UUID effectively identifies the home that produced its data, but no guarantees are made that the UUID was used by a single home, or if any data it is associated with is forged.*

### Data deletion

Data deletion is performed through the DELETE method on `/data`.

Completely delete data associated with an UUID.
This is the operation behind the Forget/Opt-out feature.
The users have the right to delete all records from the platform at any time, provided they have the UUID that they uploaded the data by.

This operation can be executed publicly on any UUID, but no feedback is transmitted on whether or not the UUID existed previously on the platform.
Although, if the operation fails for some reason, that is explicitly expressed.

## Errors

Some errors may be encountered, which may be the fault of the client or a problem with the server. Below are the custom errors for this API:

- **Malformed UUID (400)**: the supplied UUID in the header field `Home-UUID` is not properly formatted. This is due to the string provided in this field not matching the format of a proper UUID, which features 5 fields of hexadecimal characters
- **Bad ingest decoding (400)**: the request body provided, which includes the home data, is not encoded, compressed, or in the format required by the application. Make sure that it is a JSON object, is encoded using UTF-8 encoding, and is finally compressed using `zlib`. The strucure of the JSON object itself does not matter for in this error to occur

## Possible issues

Below is a list of possible issues that may be found when communicating with this API, where the fault may not be apparent at first:

- the header parameter `Home-UUID` requires the use of a dash `-` and not an underscore `_`, even though the Swagger documentation of this API presents it with one (as `Home_UUID`). If the parameter is passed with an underscore, the API will throw an error claiming that that exact parameter, with the underscore, wasn't provided
