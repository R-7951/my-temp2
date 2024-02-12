---
title: "User Application"
layout: post
date: 2023-10-09 09:52:18 -0500
categories: [FXR90,2. Advanced]
tags: []
order : 3
math: true
mermaid: true
---



## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


# Overview of DA App

## DA (Data Analytics) App

DA App enables edge computing advantage in the reader. DA App allows advanced filtering and analytics to be done before sending the tag data from the reader. 

The following are the core functions:

 - Filter, Analyze and act upon data 
 - Manage and control the reader via REST 
 - Implement Business logic and 
 - Control GPO

The reader supports hosting embedded applications developed in Python & Node.js.

# Python DA apps
The Data Analytics App can be implemented as Python using Zebra IOTC Python module available in the reader.

## ZIOTC Python Module
The Python module **pyziotc** abstracts the Zebra IoT Connector and allow the script to filter, analyze and act on the data.

The flow of the script is as follows:

 1. Define a new callback function.
	```
    def new_msg_callback(msg_type, msg_in):
    	    pass
	```
 2. Open pyziotc module. Establish connection between Python and IoT Connector
	```
    import pyziotc
    ziotcObject = pyziotc.Ziotc()
	```

 3. Register callback function. This function is called when new message arrives.
	```
    ziotcObject.reg_new_msg_callback(new_msg_callback)
	```
 
 ### New Message Callback
 
 The user-defined new message callback function is the core of the script. The callback definition must contain 2 parameters, msg_type and msg_in. The function can then process the input message and send out an output message based on the input. All messages shall be Python  [bytearrays](https://docs.python.org/3/library/stdtypes.html#bytearray)

For example, the following callback function, only sends out a data message if the beginning of the tag’s ID begins with a prefix.

```
prefix = "38"
def new_msg_callback(msg_type, msg_in):
    if msg_type == pyziotc.MSG_IN_JSON:
        msg_in_json = json.loads(msg_in.decode('utf-8'))
        tag_id_hex = msg_in_json["data"]["idHex"]

    if tag_id_hex.startswith(prefix):
        ziotcObject.send_next_msg(pyziotc.MSG_OUT_DATA, msg_in)
```

### Input Message Types
The new message callback will be called each time a new message arrives from the Radio Control. The incoming msg_type is a constant defined in the ziotc.py module. The currently available value for incoming messages is:


```
MSG_IN_JSON = 0
MSG_IN_GPI = 6        
```
### Output Message types

Messages can be sent to the Reader Gateway and off the reader by calling:

```
ziotcObject.send_next_msg(msg_type, msg_out)
```
msg_type is a constant defined in the ziotc.py module. The currently available value for outgoing messages are:

```
MSG_OUT_DATA= 3
MSG_OUT_CTRL= 4
MSG_OUT_GPO = 5
```
The **MSG_OUT_DATA** type will send the message to the Reader Gateway to output on the Data Interface. The **MSG_OUT_CTRL** type will send the message to the Reader Gateway to output on the Management Event Interface. The **MSG_OUT_GPO** type will send the control the GPO/LED.


### Data Analytics Passthru

In addition to processing data from the Radio Control and passing new, or filtered, data through to the Reader Gateway, messages can be passed through the Reader Gateway directly to the Data Analytics script. These messages can be sent to the Reader Gateway via the Management Command/Response interface. In order to process the passed through messages, a pass through callback must be defined and registered with the ZIOTC object.

The pass through callback takes in a single parameter. It is a bytearray of the incoming message.

> The function must return a response message that is also a bytearray.

This functionality enables a mechanism to configure the Data Analytics Script. The following code snippet continues the example above and allows for a single command “prefix” to be sent to the script. If only “prefix” is sent, the script returns the current value the script is using to filter tags. If prefix is followed by another string, it will update the “prefix” to the new value.

```
def passthru_callback(msg_in):
    global prefix
    parts = msg_in.decode('utf-8').split(" ")
    print(parts)

    if parts[0] == "prefix":
        if len(parts) == 1:
            response = "prefix set to {}".format(prefix)
            response = bytearray(response.encode('utf-8'))
        else:
            prefix = parts[1]
            response = "prefix set to {}".format(prefix)
            response = bytearray(response.encode('utf-8'))

    else:
        response = b"unrecognized command"

    return response

ziotcObject.reg_pass_through_callback(passthru_callback)
```

### Management and Control of Reader
DA apps can manage and control the reader using the REST interface described [here](../../api_ref/local_rest/index.html#local-rest-apis). Reader bundles the python `http` module which can be used to call local rest API.

```python
import http.client
from http.client import HTTPConnection

c=HTTPConnection("127.0.0.1")
c.request('GET', '/cloud/status')
res = c.getresponse()
data = res.read()
print(data)
```
### Controlling GPO and LED
DA apps can control the reader GPOs and LEDs using the `send_next_msg` method. LED can be controlled by sending the below payload in `send_next_msg` method.

```python
{"type":"LED","color":"RED","LED":3}
```

    Only the App LED (LED 3) can be controlled by the user app.

The sample below changes the color of the LED preiodically.

```python
import pyziotc
import json
import time

led_green_msg = bytearray(json.dumps({"type":"LED","color":"GREEN","led":3}), "utf-8")
led_yellow_msg = bytearray(json.dumps({"type":"LED","color":"AMBER","led":3}), "utf-8")
led_red_msg = bytearray(json.dumps({"type":"LED","color":"RED","led":3}), "utf-8")

z = pyziotc.Ziotc()
while True:
    time.sleep(1)
    z.send_next_msg(pyziotc.MSG_OUT_GPO, led_green_msg)
    time.sleep(1)
    z.send_next_msg(pyziotc.MSG_OUT_GPO, led_red_msg)
    time.sleep(1)
    z.send_next_msg(pyziotc.MSG_OUT_GPO, led_yellow_msg)  
```

Controlling GPO can be achieved by sending the below json object in the payload for `send_next_msg` method.

```python
{"type":"GPO","state":"HIGH","pin":1}
```

The sample below toggles the GPO preiodically.

```python
import pyziotc
import json
import time

gpo_high_msg = bytearray(json.dumps({"type":"GPO","state":"HIGH","pin":1}), "utf-8")
gpo_low_msg = bytearray(json.dumps({"type":"GPO","state":"LOW","pin":1}), "utf-8")

z = pyziotc.Ziotc()
while True:
    time.sleep(1)
    z.send_next_msg(pyziotc.MSG_OUT_GPO, gpo_high_msg)
    time.sleep(1)
    z.send_next_msg(pyziotc.MSG_OUT_GPO, gpo_low_msg)
```
### Sending Asynchronous messages from the DA app
DA apps can also send asynchronous messages via the management events endpoint of the `IoT  Connector`. The sample below illustrates sending version message at startup.

```python
import pyziotc
import json
import time

version_msg = bytearray(json.dumps({"version": 1, "time": time.now()}), "utf-8")

z = pyziotc.Ziotc()
z.send_next_msg(pyziotc.MSG_OUT_CTRL,version_msg)
```

## Sample Python DA App
Sample Python DA app that filters data and controls LED periodically.  Also sends an asynchronous message via management events whenever LED changes color.

```python
import pyziotc
import json

prefix = "43"
def passthru_callback(msg_in):
    global prefix
    parts = msg_in.decode('utf-8').split(" ")
    print(parts)

    if parts[0] == "prefix":
        if len(parts) == 1:
            response = "prefix set to {}".format(prefix)
            response = bytearray(response.encode('utf-8'))
        else:
            prefix = parts[1]
            response = "prefix set to {}".format(prefix)
            response = bytearray(response.encode('utf-8'))
    else:
        response = b"unrecognized command"

    return response

def new_msg_callback(msg_type, msg_in):
    if msg_type == ziotc.ZIOTC_MSG_TYPE_TAG_INFO_JSON:
        msg_in_json = json.loads(msg_in.decode('utf-8'))
        tag_id_hex = msg_in_json["data"]["idHex"]

        if tag_id_hex.startswith(prefix):
            ziotcObject.send_next_msg(zitoc.ZIOTC_MSG_TYPE_DATA, msg_in)

ziotcObject = pyziotc.Ziotc()
ziotcObject.reg_new_msg_callback(new_msg_callback)
ziotcObject.reg_pass_through_callback(passthru_callback)

led_green_msg = bytearray(json.dumps({"type":"LED","color":"GREEN","led":3}), "utf-8")
led_yellow_msg = bytearray(json.dumps({"type":"LED","color":"AMBER","led":3}), "utf-8")
led_red_msg = bytearray(json.dumps({"type":"LED","color":"RED","led":3}), "utf-8")

#The below formats for async messages are for illustration only. it can be anything the app wants.
async_msg_red = bytearray(json.dumps({"source":"Sample DA app","message":"LED set to RED"}), "utf-8")
async_msg_green = bytearray(json.dumps({"source":"Sample DA app","message":"LED set to GREEN"}), "utf-8")
async_msg_yellow = bytearray(json.dumps({"source":"Sample DA app","message":"LED set to YELLOW"}), "utf-8")

while True:
    time.sleep(1)
    z.send_next_msg(pyziotc.MSG_OUT_GPO, led_green_msg)
    z.send_next_msg(pyziotc.MSG_OUT_CTRL, async_msg_green)
    time.sleep(1)
    z.send_next_msg(pyziotc.MSG_OUT_GPO, led_red_msg)
    z.send_next_msg(pyziotc.MSG_OUT_CTRL, async_msg_red)
    time.sleep(1)
    z.send_next_msg(pyziotc.MSG_OUT_GPO, led_yellow_msg)
    z.send_next_msg(pyziotc.MSG_OUT_CTRL, async_msg_yellow)

```
# Node.js DA App

## Overview 
	
The Data Analytics functionality can be implemented as a Node.js script using the Node.js ZIOTC (Zebra IOT Connector) module available in the reader.

The Node.js module **ziotc.node** abstracts the Zebra IoT Connector and allow the script to filter, analyze and act on the data. 

The minimal flow of the script would be:




