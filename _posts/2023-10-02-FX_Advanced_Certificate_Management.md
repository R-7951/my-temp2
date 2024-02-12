---
title: "Certificate Management"
layout: post
date: 2023-10-09 09:52:18 -0500
categories: [FXR90,2. Advanced]
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



# Certificate Management
This section covers on how to manage the certificates on the reader. The certificates used for making secure connection between the reader and clients.

# Certificate Management
The reader provides an ability to install multiple certificates. There are mainly three types of certificates that can be installed in the reader.
## Server
This is the Main certificate.  There can only be one server certificate on the reader. The reader by default comes up with a self-signed certificate. 

A certificate installed as the server certificate is used by the reader to secure the **incoming** connections to the reader (For Ex: Reader Web UI, SSH & SFTP).

## Client
There can be more than one client certificates installed on the reader. The client certificates are typically used to secure **outgoing** connections from the reader. The certificate is the recommended type with Zebra IoT Connector interface.

## App
This type of certificate can be installed to use with user applications deployed on the reader.

> The reader supports the certificate only in PFX format.

# Manage Certificate 
The reader supports to update the certificate from secure server. The following are the secure protocols:

 - HTTPS 
 - FTPS/SFTP

## Update/ Install Certificate
The update certificate has the following input parameters:

 - Certificate Type 
 - Name Server	
 - Location (URL) & Credentials Certificate
 - PFX password
 
 ## List of Installed Certificate
 The reader gets the list the installed certificates. 
 Each certificate contains the subject name, Issuer, Friendly Name, Type, validity, serial No, installed date.
 
 ## Delete Certificate
 The certificate can be deleted from the reader.

## Refresh
This option allows to fetch the certificate update from the secure server.




