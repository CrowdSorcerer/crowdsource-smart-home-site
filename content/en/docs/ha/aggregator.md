---
title: "Aggregator"
description: ""
lead: ""
date: 2022-06-25T11:52:48+01:00
lastmod: 2022-06-25T11:52:48+01:00
draft: false
images: []
menu:
  docs:
    parent: "ha"
weight: 999
toc: true
---

To collect, clean, and send the smart house data, this custom Home Assistant Integration is available through the Home Assistant Community Store.

# Data Collection

## Overview

The Aggregator starts collecting data when installed, and allows the user to control from which sensors they wish to send data. This selection is done during install and can be done at any point after, as well. 

The filter used is in the form of a whitelist, in order to give the user the most control over what data is collected.

## In More Detail

### **Sensor Whitelisting**

 **Sensors** are filtered using a whitelist, so that new sensors that are added after the aggregator is installed aren’t automatically included. This whitelist can be accessed at any time via the Options area in the Integrations screen, and can be altered an unlimited amount of times. 

Home Assistant is queried every time this configuration takes place, in order to provide an up-to-date sensor list, with selected options transferring over between configurations, so the user doesn’t need to recall which sensors were previously selected.

The chosen sensors are stored in a manner that restarting Home Assistant won’t wipe this list, which enables the Aggregator to be set-and-forget, not requiring further user input, unless they want to alter anything.

The user is also given options to select “None” and “All” sensors - these options override any custom selections done, and will either send the data from every sensor or stop sending data at all. The “None” option will take precedence over “All”, sending no data is both are selected.

### Data Collecting

On installation and restarts, a random time between 0-6AM is picked. From then on, the Aggregator will run at this set time, once a day. It runs during the night so that any possible impact on the users’ network or Home Assistant installation is minimized, and at a random time to avoid overloading the Data Lake with a lot of data to process at once, from multiple sources.

To collect data, the aggregator will query Home Assistant’s internal database, using HA’s own methods to do so - no direct data manipulation is done - and gather all state changes (Events, polls…) generated since the last time it ran. In case it’s the first time running after installation (or a restart), it will get the states generated from 24h prior.

## Behind the Curtain - Technical Information

(We will explain in detail here how exactly things work - If you’re not interested in the code itself, you may skip this part)

### Sensor Whitelisting

The whitelist is stored in a **config entry**, created when the Aggregator is first installed and later updated when the user alters their configuration.

As we can see below, it is saved in an entry with the name **options:**

```python
self.async_create_entry(title="options", data=data)
```

The Data is structured as a dictionary with the following entries:

- `uuid` - contains the user’s UUID, which is set only once and should never be altered
- `sensors` - the sensor list that is meant changed in the future, if the user wants to

To create this sensor list we query the internal database in the same manner as when collecting data, only here we simply collect the name of all the sensors present in the returned data, with the exception of the Aggregator itself, `sensor.crowdsourcerer` and the notifications it generates, `persistent_notification.data_collector_notification`.

Below is the snippet where the database is queried and the sensor list created

```python
# Collect the data from the internal database
raw_data = await recorder.get_instance(self.hass).async_add_executor_job(
     functools.partial(
         state_changes_during_period, start_time=start_date, hass=self.hass
     )
)
# Filter it to get the sensor names only
sensor_data = {}
for key, value in raw_data.items():
    sensor_data[key] = [state.as_dict() for state in value]

    sensors = ["All", "None"] + [
        key
        for key in sensor_data
        if (
             key != "sensor.crowdsourcerer"
             and key != "persistent_notification.data_collector_notification"
         )
    ]
```

### Data Collecting

**Scheduling**

To run at a set, randomized time, the aggregator makes use of `async_track_time_change`, which will callback the function that does all the heavy lifting, `async_collect_data`, as can be seen below:

```python
from homeassistant.helpers.event import (
    async_track_time_change,
)
self.random_time = [randint(0, 6), randint(0, 59), randint(0, 59)]
schedule = async_track_time_change(
    self.hass,
    self.async_collect_data,
    self.random_time[0],
    self.random_time[1],
    self.random_time[2],
    )
```

Once this function is called, it will first check if the `last_ran` attribute is set, otherwise it will be set to 24h before the current date and time.

Then, we fetch the sensor whitelist from the **configuration entry:**

```python
# Initialize the whitelist
allowed = []
# Get all config entries and find the Aggregator's entry
entries = self.hass.config_entries.async_entries()
for entry in entries:
entry = entry.as_dict()
if entry["domain"] == "data_collector" and entry["title"] == "options":
		 # Retrieve the data from the entry
     self.uuid = entry["data"]["uuid"]
     allowed = entry["data"]["sensors"] 
     break
```

```python
allowed = []
        entries = self.hass.config_entries.async_entries()
        for entry in entries:
            entry = entry.as_dict()
            if entry["domain"] == "data_collector" and entry["title"] == "options":
                self.uuid = entry["data"]["uuid"]
                allowed = entry["data"]["sensors"]
                break
```

To collect the data we make use of `homeassistant.components.recorder.history`'s function, `state_changes_during_period`, using `last_ran` as starting point, and the current date/time as ending point.

The relevant code snippet for this is this:

```python
from homeassistant.components.recorder.history import state_changes_during_period
raw_data = await recorder.get_instance(self.hass).async_add_executor_job(
		functools.partial(
	    state_changes_during_period, start_time=start_date, hass=self.hass
		)
)
```

To avoid blocking the entire home assistant installation while carrying out this operation, this is done via an **executor job.**

Then, the retrieved data is filtered with base on the whitelist, and converted to a dictionary instead of objects for ease of work.

```python
for key in raw_data.keys():
		if (
       key not in allowed and "All" not in allowed # allowed is the whitelist
    ) or key == "sensor.crowdsourcerer": 
       filtered_data.pop(key)

for key, value in filtered_data.items():
    sensor_data[key] = [state.as_dict() for state in value]
```

After this, the data is further filtered in the next section.

# Data Anonymization

## Overview

After being collected, the data is filtered in multiple ways to censor as much Personally Identifying Data (From now on, referred to as PII) from the data, **before** it leaves the users’ local installation.

The data that is filtered is:

- Credentials
    - In username: <username> password: <password> formats
- Credit card numbers
- Email Addresses
- Phone Numbers
- Postal Codes - Portuguese and British formats
- Twitter handles
- URLs
- IP Addresses
- GPS Coordinates
- Unique IDs

The data is also prepared to be sent to the data lake, and sanitized to remove characters that cannot be present.

## In More Detail

Data anonymization is by far the most time-consuming part of the Aggregator’s operations, with its performance *very* dependent on the size of the collected data. Where it is nearly instantaneous in processing small amounts of data, larger sets can take multiple minutes to process.

To filter out data, three methods are used.

For reference, the data is organized as a dictionary(Also known as Key-Value) structure; Each field has a name, the key, to which it associates a piece of data, the value. Each sensor typically has a high number of key-value pairs inside.

### **Key Based Filtering**

The data is filtered with basis on the **key** - if this key is contained in a list of **forbidden keys**, the value associated with it is **scrubbed**, and replaced with {{REDACTED}}. Currently the forbidden keys are GPS coordinates (Latitude, Longitude) and user IDs. Due to the unpredictable nature of the data, this method is not very effective, since key names can be very varied.

### **Scrubadub**

[Scrubadub](https://pypi.org/project/scrubadub/) is a Python library used to remove PII from text. It is what cleans up most of the data, and it uses a large number of filtering methods to do so. Unfortunately, some of its filters are UK/US centric and as such don’t suit our purposes, requiring alternate implementations. Most of what scrubadub cleans is usually values, but keys can, and will, also be cleaned if there’s PII contained in them.

Originally there was the option to filter out people’s names both in english and portuguese, as well as country and portuguese location names, but this had to be scrapped due to **severe** performance issues - The operation could take over an hour with a sizeable data amount, and Home Assistant would *not* handle it well. 

### **Pattern Matching**

To patch some holes in Scrubadub coverage, pattern matching is used to detect and replace certain patterns in the data, such as IP Addresses and portuguese postal codes. These kinds of data use a very rigid format, and as such can be quickly removed using **Regular Expressions.** This is the most efficient and effective method to filter data, but since it can only be used with strict patterns, its scope is fairly limited - it can easily detect that there’s a sequence of nine numeric characters (A phone number), but not that someone’s name is present in the data.

### Data Sanitization

Due to technical limitations on the Data Lake, there’s a certain set of characters that **cannot** be present in the data inserted. However, some of these characters are essential to the data structure, and as such, removing them isn’t as straightforward as one would think. 

## Behind the Curtain - Technical Information

### A Custom Scrubadub

The Aggregator uses a custom version of Scrubadub, called [cs-scrubadub](https://pypi.org/project/cs-scrubadub/), which is a slimmed down version of Scrubadub, while keeping all the needed functionality intact. This was needed due to severe version conflicts in the dependencies with Home Assistant itself that would completely block the Aggregator from even installing, outside a development environment. Since these dependencies were needed only for features that are not used in the Aggregator, they were scrapped.

### Key Based Filtering

Since the data collected doesn’t have a strict structure, it is quite difficult to iterate over - It can have nested dictionaries and lists. So, for this we needed a recursive method to iterate over all the data, in order to find blacklisted keys and redact their values.

```python
def custom_filter_keys(data):
		"""Filters based on key names"""
    if isinstance(data, dict):
		    for key in data:
		        if key in FILTERED_KEYS:
		            data[key] = "{{REDACTED}}"
            if isinstance(data[key], (dict, list)):
                custom_filter_keys(data[key])

		elif isinstance(data, list):
		    for it in data:
		        custom_filter_keys(it)
		return data
```

### Scrubadub

The inner workings of Scrubadub can be consulted in its’ technical documentation, [available here](https://scrubadub.readthedocs.io/en/stable/index.html#).

Our usage of scrubadub is mostly using the defaults, only using a custom post processor to replace the found PII with a custom string, `{{REDACTED}}`.

### Pattern Matching

Used to filter out IP Addresses and Portuguese Postal Codes, the **Regular Expressions** are the following:

```python
import regex as re
def custom_filter_reg(data):
		"""Filters based on Regex Expressions"""
		data = re.sub(r"\ (?:[0-9]{1,3}\.){3}[0-9]{1,3}\b", "{{REDACTED}}", data)
		data = re.sub(r"\d{4}[\-]\d{3}", "{{REDACTED}}", data)
		return data
```

### Data Sanitization

To sanitize the data, python’s string manipulation capabilities are used, in a mostly straightforward manner:

```python
data = (
        data.replace(".", "_")
        .replace("<", "_")
        .replace(">", "_")
        .replace("*", "_")
        .replace(":", "_")
        .replace("#", "_")
        .replace("%", "_")
        .replace("&", "_")
        .replace("\\\\", "_")
        .replace("+", "_")
        .replace("?", "_")
        .replace("/", "_")
    )
data = data.replace(" _ ", ":") 
data = re.sub(r"(?<=\d)_(?=\d)", ".", data)
```

# Other functionalities

## Compression & Network

### Overview

To shrink the size of the data sent, and thus reduce the time it may take to transfer as well as using up less of the volunteer’s home network, the data is compressed.

This data is sent to the Data Lake via a POST HTTP request, with a header containing the user’s UUID, to allow deletion at the user’s request, in the future. 

### Technical Information - Data Transfer to API

The data collected by the aggregator is converted into a JSON format, then encoded into bytes in utf-8 and finally compressed using Zlib.

- **Why Zlib?**
    
    Zlib was chosen as the compression algorithm to use since it was the best-performing out of three that were tested. This algorithm was the one who yielded the best results both in speed and final file size (smaller is better)
    
   <img src="aggregator_zlib.png" alt="Comparison between multiple possible compression algorithms.">
    
    *(The number below each algorithm is how long the compression took, in seconds)*
    

It is sent to the Ingest API via POST request:

```python
r = requests.post(
            api_url,
            data=local_data,
            headers={
                "Home-UUID": user_uuid,
                "Content-Type": "application/octet-stream",
            },
        )
```

# Extra Technical Information

## File Structure

**__init__.py** → Contains simple setup, first thing that’s loaded by Home Assistant (Entrypoint to the integration)

**config_flow.py →** An important file, defines how configuration flows work.

class **ConfigFlow** : controls what happens when the integration is added to Home Assistant, as well as handling the callback to open the options menu, after installation is complete.

function **async_step_user** → Handles the initial configuration step, which is getting the allowed categories of data to send from the user and generating an uuid, which are saved (persistently) via home assistant’s **configuration entries,** with the title “options” in the ‘data_collector” domain.

<aside>
/!\ Pay special attention to the variable **user_input,** which is where the data entered by the user during configuration is stored. When this isn’t None, the configuration has already taken place, and we must save the user’s input in a **config entry.** Otherwise, regular configuration takes place

</aside>

This config entry can be accessed later in the Entity via this code snippet below.

```python
entries = self.hass.config_entries.async_entries()
for entry in entries:
	entry = entry.as_dict()
  if entry["domain"] == "data_collector" and entry["title"] == "options":
			# Insert your code here
```

class **CollectorOptionsFlow** : controls what happens when the user opens the ‘Configure’ panel, after the integration has been installed.

- The flow for this is the exactly same as when installing the integration, save for generating the UUID, which is kept the same.

**sensor.py →** Contains the code for the integration itself, as well as utility functions to compress and send data over a network, and various setup steps related to Home Assistant functionality.

**Entity →** The actual integration is an object of the type Entity. The main logic is contained in an asynchronous function that’s called when the local HA clock hits a certain time in the day, as explained at the start of this document.
