---
title: "Settings"
layout: post
date: 2023-10-05 09:52:18 -0500
categories: [FXR90,1. Basic]
tags: []
order : 5
math: true
mermaid: true
---



## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---



# Settings

The reader supports REST/MQTT interface to control and manage the reader. 

## Control
The control interface primarily manages the RFID operations. 

| Function | Description | REST API | Notes | 
|--|--|--|--|
| start	|Starts RFID operation | PUT /cloud/start| By default, reads the tags in the range of antenna. Operating Mode Controls the RFID operation. Allows to read, write, kill, lock the tags.
| stop	|Stops RFID operation | PUT /cloud/stop| Stops on-going RFID operation.
| get mode	|Gets the current operating mode | GET /cloud/mode| Retrieves the current operating mode.
| set mode	|Sets the reader's operating mode | PUT /cloud/mode| Updates the operating mode.

### Operating Mode
This section covers the details of the operating mode and other relevant parameters. 


 1. Mode - Set the readers with optimal settings depending on the use case.

	- **Simple**: Report all unique tags in the range of reader, for all the connected antennas by default. 
	- **Inventory**: Report all unique tags for each antenna in periodic intervals (default 1min) 
	- **Portal**: Report all unique tags for each antenna after the GPI start trigger. Once stops reading, waits for another GPI trigger. (Default set to wait on GPI1 for Low to start and 3s duration stop trigger) 
	- **Conveyer**: Report all unique tags for each antenna. 
	- **Custom**: Provides an option to adjust or set parameters for advanced used cases. 


 2.  Environment - Specify the deployment environment in which the reader operates. 
	 - **Low interference**: Interference is very low or only occurs for a very short time, typically a single reader.
	 - **High interference (Default)**: Operating in the presence of other readers, typically muti or dense readers.
	 - **Very high interference**: Number of the readers is greater than the number of available channels or the deployed readers are in very close proximity to each other.
	 - **Auto detect**: Automatically access the environment and adjust.
	 - **Demo** : Demonstrate maximum read performance. Assumes no other readers in the environment.

 3. Antennas - select the antenna for the RFID operation.
 4. Transmit Power - Control the transmit power for the antenna.
 5. Tag Reporting 
	 - report unique tags
	 - report tag once for every __ seconds for all the antennas.
	 - report tag once for every __seconds in each antenna.
	 
 6. C1G2 Commands
	- Select
	- Query
	- Access 
 7. Pre-selection (Cellular band filter) - Used for external non-RFID noise cancellation. 


## Management

This section includes the core functions related to management of the reader.

| Function | Description | REST API | Notes | 
|--|--|--|--|
| version	|Gets reader software version | GET /cloud/version| 
| status	|Gets the reader status | GET /cloud/status| Antenna Connect status, CPU/Memory utilization, Temperature, up time, etc.
| reboot	|Reboot the reader | PUT /cloud/reboot| 
| update software	| Updates software on the reader | PUT /cloud/os| Allows to update from HTTPS/SFTP/FTPS URL
| get network	| Gets Network interface configuration | GET /cloud/network| 
| set network	| Sets Network interface configuration | PUT /cloud/network|





