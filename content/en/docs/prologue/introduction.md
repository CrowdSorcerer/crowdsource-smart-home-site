---
title: "Introduction"
description: "Platform for crowdsourcing smart home data"
lead: "Platform for crowdsourcing smart home data"
date: 2020-10-06T08:48:57+00:00
lastmod: 2020-10-06T08:48:57+00:00
draft: false
images: []
menu:
  docs:
    parent: "prologue"
weight: 100
toc: true
---

The main objective of this project is to create a platform to gather through a crowdsourcing mechanism anonymous information from homes. The base of this project is be the Open Source Software (OSS) project [Home Assistant](https://home-assistant.io) and the [Portuguese community of users](http://cpha.pt).

## The team

We’re a team of four students in our 3rd year of the Informatics Engineer Bachelor’s degree at University of Aveiro.

- ***Coordinator:*** Diogo Gomes

- **Team Leader:** Martinho Tavares
- **Frontend Dev**: Diogo Monteiro
- **Backend Dev:** Camila Fonseca 
- **DevOps Engineer:** Rodrigo Lima

## Components

The project involves the development of 3 main components: a Data Lake, a Dashboard and a custom component for Home Assistant. Since we will be dealing with information that can be considered sensible, unless it is not anonymized, a great deal of importance will go towards providing informed decision making (choosing what to share), right to be forgotten and anonymization techniques.

- **Data Lake**: storage, intake and exportation of heterogeneous information (mostly JSON docs). Will include 3 APIs:
    - **Ingest API**: publicly exposed, for data ingestion and deletion of data (opt-out feature)
    - **Query API**: consumption by the Dashboard in near realtime *(role currently taken by Prometheus)*
    - **Export API**: data dumping in CKAN compliant formats
- **Dashboard**: web application for information relating to the health of the platform and status of the data, such as uptime and number of volunteers
- **Home Assistant Custom Component**: component for the Home Assistant platform to aggregate and send the authorized data to our Data Lake. It is divided in two sub-components:
    - **Aggregator**: component that obtains data from the available integrations and aggregates them, sending it to the Data Lake
    - **Dashboard card**: interface for interaction with the volunteer, to collect an informed consent to data collection and allow customization of which data to send

## Links

Below are links related to the project's development.

- [Organization's GitHub](https://github.com/CrowdSorcerer)
- [Jira project](https://martinhotav.atlassian.net/jira/software/projects/CSHD/boards/1/roadmap)
