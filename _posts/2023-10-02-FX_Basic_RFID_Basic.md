---
title: "RFID Basic"
layout: post
date: 2023-10-05 09:52:18 -0500
categories: [FXR90,1. Basic]
tags: []
order : 2
math: true
mermaid: true
---



## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


# RFID Basics
  

RFID (Radio-frequency identification) technology helps to identify assets/items labelled with RFID tags.  The frequency range used for UHF (Ultra High Frequency) RFID is 860MHz to 960MHz. The tags are read in the range of antenna. The range varies depends on the antenna type used for the use cases. 

The RFID Reader and tag follows the standard "EPC™ Radio-Frequency Identity Protocols
Class-1 Generation-2 UHF RFID Protocol for Communications at 860 MHz – 960 MHz" defined by GS1 EPCglobal.

https://www.gs1.org/sites/default/files/docs/epc/uhfc1g2_1_2_0-standard-20080511.pdf
    
The following section covers the major building blocks of RFID Reader solution.

# Building Blocks 
  

## RFID Reader
  

The RFID reader transmits the RF energy via antenna. The tag sends back his identification detail. The software stack in the reader bridges to the outside world applications.  

The core function of the reader is Modulate/transmit and receive/demodulate a sufficient set of the electrical signals defined in the signaling layer C1G2 specification to communicate with conformant Tags and conform to all local radio regulations.


## Antenna
  
The antenna is connected to the reader via co-axial cable.   The antenna must be compliant to UHF Frequency Range (860MHz to 960MHz), There are variety of antennas available to support various use cases and read range to cover. 

## Tags

The tag modulates a backscatter signal only after receiving the requisite command from an Interrogator and conform to all local radio regulations.

Typically, the tags are attached to assets or items which needs to be tracked. 

The tag must meet the C1G2 specification and implement all the mandatory commands.

There are variety of tags available and can be selected depends on the use cases. 

### Passive Tag
The tag does not have a battery and harvest the energy transmitted from the reader via antenna.

### Battery Assisted Passive (BAP) tag
The tags are designed to have a battery support which helped to read in the long range.

> For more details, please refer the section "Tag selection, inventory, and access" in GS1 C1G2 specification document.

### Memory Map
The tag memory is logically separated into four memory banks.

|Memory Bank|Data|
|--|--|
|Reserved| Kill (First two words) and access passwords (3&4th word)
|EPC| Identifies the asset/item beginning from 4th byte or 2nd word )
|TID| Tag manufacturer specific information (Tag Model, serial number). PermaLocked during tag manufacturing. 
|USER| Optional. Allows user-specific data storage.

### Sessions and inventoried flag

Tag support four sessions (S0, S1, S2, S3). 
Tags participate in one and only one session during an inventory round. 
Tag maintains an independent **Inventoried** flag for each session. Each of the four **Inventoried** flag has two values, denoted by A and B.

### Selected flag (SL)

Tag has selected flag (**SL** flag). The reader may assert (SL) or deassert (~SL) the SL flag during tag population selection.

### Tag persistence

|Flag|Persistence|
|--|--|
| S0 **inventoried** flag|Indefinite if tag energized|
| S1 **inventoried** flag |500ms to 5s if tag energized|
| S2 **inventoried** flag |Indefinite if tag energized|
| S3 **inventoried** flag|Indefinite if tag energized|
| Selected **SL** flag|Indefinite if tag energized|



# Managing tag population

The reader manages the tag populations using the three basic operations as shown below:
```mermaid
graph LR
A[Select] -->B[Inventory] --> C[Access]
```
## Select

The reader Selects the tag population based on the following:

 - Target indicates whether Tag's **SL** or **inventoried** flag
 - Action indicates whether matching tags
	 -  modify **SL** (Assert SL or deassert SL) or **inventoried** flag (A or B)
 - Memory Bank (EPC, TID or User Memory)
 - Start Location, Length, Mask (bit string that tag compares)
 
# Tag Response to Action parameter:

|Action|Matching|Non-Matching|
|--|--|--|
| 0 |assert **SL** or **inventoried** -> A |deassert **SL** or **inventoried**->B|
| 1 |assert **SL** or **inventoried** -> A |do nothing|
| 2 |do nothing |deassert **SL** or **inventoried**->B|
| 3 |negate **SL** or **inventoried** (A->B, B->A)|do nothing|
| 4 |deassert **SL** or **inventoried** ->B |assert **SL** or **inventoried** ->A |
| 5 |deassert **SL** or **inventoried** ->B |do nothing |
| 6 |do nothing |assert **SL** or **inventoried** ->A |
| 7 |do nothing |negate **SL** or **inventoried** (A->B, B->A) |

## Inventory
The Inventory operation determines which tags to be participated based on the following parameters:
 - Sel - which tags to respond based on **SL** flag 
	 - 0: ALL
	 - 1: ALL
	 - 2: ~SL
	 - 3: SL	 
 - Session - choose the session for the inventory round.
	 - 0: S0
	 - 1: S1
	 - 2: S2
	 - 3: S3
 - Target (**inventoried** flag)
	 - 0: A
	 - 1: B
 - Q - Tag population in the range of antennas.

## Access
The reader implements all the required commands read, write, kill and lock. 

BlockWrite, BlockErase and BlockPermalock are optional. This specific to tag capability supported by the tag Vendor.

### Read
The reader can read part or all of Tag's Reserved, EPC, TID or User Memory. 
The read command has the following parameters:
 - MemBank - Memory Bank to read
	 - 0: Reserved
	 - 1: EPC
	 - 2: TID
	 - 3: User Memory
 - WordPtr - Starting location
 - WordCount - Length of the data to be written
 - Data - 16-bit word to be written

### Write
The write allows the reader to write a word in Tag's Reserved, EPC, TID, User memory. 
The write command has the following parameters:

  - MemBank - Memory Bank to write
	 - 0: Reserved
	 - 1: EPC
	 - 2: TID
	 - 3: User Memory
 - WordPtr - Starting location
 - WordCount - Length of the data to be read

### Kill
The Kill allows the reader to permanently disable a tag. The tag will not be reported any more.  The tag must be set with Kill password.

The kill command has the following parameter:

 - Password - kill password 

### Lock
The lock command allows the reader to:

 - Lock individual passwords, preventing or allowing subsequent reads and writes of that password
 - Lock individual memory banks, preventing allowing subsequent writes of that bank
 - Permalock (make permanently unchangeable) the lock status for a password or memory bank

The lock command has the following parameter:

 - Mask - specifies passwords & memory banks to lock
 - Action 
	 - read/write, permalock for passwords
	 - write, permalock for memory banks
