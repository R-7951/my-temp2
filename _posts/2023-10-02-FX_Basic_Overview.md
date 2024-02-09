---
title: "Overview"
layout: post
date: 2023-10-07 09:52:18 -0500
categories: [FXR90,1. Basic]
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


# What is IoT Connector?

Zebra **IoT Connector** is new modern simplified interface to manage, configure and control the RFID reader.  The interface is built on using web technologies such as REST, MQTT, WebSocket, JSON formatted data structures that can be easily integrated by any application running on the cloud or on-premises. 

The Key features of the IoT connector are listed below.

 - Interfaces
	 - API/Command: REST, MQTT 
	 - Events Streaming:
		 - Tag Data: MQTT, WebSocket, HTTP, TCP/IP (Secured/Non-secured)
		 - Management Events: MQTT, WebSocket, HTTP
 - RFID operation and configuration made simple
	 - Operating Mode to cater various use cases
	 - Environment configuration specific to deployment setup
 - Cloud support: AWS, Azure
 - DA (Data Analytics) App to run inside the reader to support customer apps
 - Batching and retention (retain 150000 tag events)
 - Easy to define rule-based mechanism to control GPO 
 

# System Overview
 
Image to be embedded here.
 
# Interfaces
The Zebra IoT Connector supports the various interfaces. The interfaces can be configured via Endpoint configuration. 

## Management
The management interface allows to perform management functions on the reader. The primary functions are:

 - Software update
 - Get / Set reader config
 - Enterprise Reset
 - Reboot
  
This interface can be configured using either MQTT or REST API.

## Control
The Control interface helps to control RFID operations on the reader such as start, stop and set the operating mode.

This interface can be configured using either MQTT or REST API.

> When configured to use MQTT, command and respond topics must be specified to perform management function.
> 
> Management/Control interface is based on command/response model (Synchronous). 
 

## Events 
The reader sends an asynchronous message on this interface.  

When MQTT end point configured, the management events topic must be configured to consume events from the reader.

### Data (Tag Data Events)
This interface streams the RFID tag data events to the configured endpoint.  IoT connector supports two separate data endpoints and can be configured to any of the following endpoints:
 1. MQTT
 2. WebSocket
 3. HTTP post
 4. TCP/IP (reader acts as server)

For MQTT, the tag event topic must be configured to consume the tag events.

To handle network connection related issues, the data interface implements the Batching and retention methods:

#### Batching
Reader groups multiple tag events into single event based on the configuration. Batching helps to reduce network and resource (CPU/Memory) usage. 

#### Retention
Reader buffer the tag events and stream data back to server in case of network connection issues or server failures. 

By default, retention is configured to retain most recent 150000 tag events and stream back to server at 500 tags per second.

### Management Events
The interface can be used to monitor the health and status of the reader.  This event can be configured any time. 

The reader can be configured to report the events for any of the following endpoint:

 1. MQTT
 2. WebSocket
 3. HTTP post
 
