---
title: "Lovelace Card"
description: ""
lead: ""
date: 2022-06-12T11:13:41+01:00
lastmod: 2022-06-12T11:13:41+01:00
draft: false
images: []
menu:
  docs:
    parent: "ha"
weight: 999
toc: true
---

To extend the Crowdsorcerer integration's functionality, a custom Lovelace card for Home Assistant's dashboard is available
and recommended.

## Features

### View information about sent packages

Users can see in the main screen information about the size of the packages they send, and the date they are sent.

<img src="card_main.png" alt="card's main screen">

### View assigned ID

The assigned UUID for the current Aggregator installation can be viewed. It is recommend that users save this ID in a safe place, as it is the key to delete their data should they uninstall the Aggregator.

<img src="card_view_id.png" alt="card's manage data screen, displaying UUID">

### Download last sent data

The last data to be sent can be downloaded as a JSON file, so that users may view in detail what information is collected and sent.

<img src="card_download_data.png" alt="card's manage data screen, displaying download button">

### Delete all previously sent data

Users may request to have their data deleted from the data lake, and in doing so will receive confirmation on whether the operation was completed or not.

<img src="card_delete.png" alt="card's delete data screen, displaying a prompt and buttons to proceed or cancel">

### View terms and contact information

Users can refer to questions and answers to clarify common doubts and concerns, and view contact information such as the Data Protection Officer's email address.

<img src="card_terms.png" alt="card's terms screen, displaying informative text">
