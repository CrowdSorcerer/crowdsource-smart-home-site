---
title: "Ingest API"
description: ""
lead: ""
date: 2022-05-11T17:10:20+01:00
lastmod: 2022-05-11T17:10:20+01:00
draft: true
images: []
menu:
  docs:
    parent: ""
weight: 999
toc: true
---
Ingest API for data ingestion into the data lake.

This API should be used solely by the Home Assistant Aggregator component, but there are no restrictions on where the requests come from, since those details can easily be forged.

**Default link**: '[https://smarthouse.av.it.pt/api/ingest](https://smarthouse.av.it.pt/api/ingest)'

*Note: this documentation is also present as a Swagger documentation page on* [https://smarthouse.av.it.pt/api/ingest/ui](https://smarthouse.av.it.pt/api/ingest/ui)

# Endpoints

All endpoints are relative to `/api/ingest`.

| Endpoint | Method | Header parameters | Path parameters | Query parameters | Request body |
| --- | --- | --- | --- | --- | --- |
| /data | POST | Home_UUID (required) | - | - | Home data (binary) |
| /data | DELETE | Home_UUID (required) | - | - | - |

Parameter definitions:

- **Home_UUID**: the UUID of the home relative to the operation (add data from home with UUID or delete all data of home with UUID). It should be in the proper UUID format.

# Operations

## Data upload

Data upload is performed through the POST method on `/data`.

Through the Home Assistant Aggregator, each home produces a data payload that is inserted into the platform as it is received.

Each of these payloads is associated with an UUID, which is stored within the Aggregator instances and is sent along with the data.
The data upload to the platform is done publicly, with no proper authentication.

*As it stands, this UUID effectively identifies the home that produced its data, but no guarantees are made that the UUID was used by a single home, or if any data it is associated with is forged.*

## Data deletion

Data deletion is performed through the DELETE method on `/data`.

Completely delete data associated with an UUID.
This is the operation behind the Forget/Opt-out feature.
The users have the right to delete all records from the platform at any time, provided they have the UUID that they uploaded the data by.

This operation can be executed publicly on any UUID, but no feedback is transmitted on whether or not the UUID existed previously on the platform.
Although, if the operation fails for some reason, that is explicitly expressed.
