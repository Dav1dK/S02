# ITxPT S02 Client Development Guide #

This is the S02 Client Development Guide. It contains general advice for writing clients toward the ITxPT S02 services. There may exist other documents with more specific advice. 

In particular [KnownIssues.md](../KnownIssues.md) describes current Issues with the S02 specifications. 

The specifications have HIGHER priority than this Guide. Both S02P00 and the particular specification must be followed. 

## Background ## 

ITxPT specifications and labelling aims at ensuring interoperability for the ITxPT services. Labelling is done for services but not for the clients using those services. And while the ITxPT S02 specifications describes the interface that must be implemented by both client and service, there is an emphasis on the service part. 

For ITxPT to provide truly interoperable services, it is important that the clients follow some best practices. This is particularly important as there are no compliance tests for client implementations. 
This document contains some best practices and advice for writing interoperable client software towards the ITxPT services. The goal is for clients to be (more) compatible with different implementations of a service and with future backwards compatible releases of the specifications. 

## Please report Issues ##

If you find issues during specification review, implementation, integration, or at any other point, please report them! An issue that should be reported certainly includes errors, but could also be something that is unclear, missing, conflicting, or just something could be improved. 

This applies not only to the specifications, but also the ancillary documentation, e.g. this Guide.

For how to do this see **How to report Issues** in [README.md](../README.md) in this repository. 

## mDNS and DNS-SD ##
ITxPT service interoperability is built on mDNS and DNS-SD. 

### Read and use the SRV and TXT record attributes ###
Most of the SRV and TXT record attributes have reasonable example values. And it is likely that most service implementation will use the examples values. However, this is not a requirement, and there are many reasons using the example value is not possible. The client should never make assumptions about the values in the SRV and TXT records; it should read the values and use those. 

### Check version ###
The TXT record attribute version lists the compatible specification version(s?). If specification version is not inside the expected range, this should, at minimum, be logged as a warning. 

### Multiple services ###
TBD – Frankly not sure how to handle this  
In the cases when more than one service of a type is registered, one registration will necessarily be available before the other one, so just checking if only one service is present is not on its own enough. 

### Changing values ###
Both the SRV and TXT records could potentially change during operation. This is probably unusual, so perhaps handling this is not essential. But each client should have a clearly defined and communicated strategy on how to (not) handle changing SRV and TXT records. 

## Reading Data ##
Reading data is the primary activity of most the ITxPT clients. All of the ITxPT services work with XML or Json data. 

### Use a real parser ###
While rolling your own it-is-just-a-few-strings parser can seem tempting if just a bit of data is wanted, it is hard to handle all variants of structured data. Or at least it will be three developers, two spec revisions, and lots of more data down the road. Using a real xml/json parser ensures that any errors is the data received, not in the client parsing of data. 

### Tolerate additional data ###
By far the most common change to the specifications is that additional data is added to an existing data structure. E.g. in GNSS Location service version 2.1.0 the new element <Fix>…</Fix> was added to the GNSSLocation Element. Apart from that, the data structure was unchanged. 

Being tolerant of additional data in whatever form will greatly improve the chances of the client being forward compatible with future versions of the specification. 

*Note:* Unfortunately the XML XSDs are not currently written to allow extra elements and attributes. This means using the xsd schemas in production *and failing if data does not validate cleanly* is probably something that should be avoided. (The MADT JSON schemas allow extra properties so they are fine.)

### Handle missing data ###
There are special cases where some expected specified data is not available. In these cases, the service may have little choice than to send less data than would strictly be expected. Either an expected element/attribute could be missing – e.g. expected element <data>…</data> is not found – or the content of an element/attribute could be missing – e.g. it is <data></data>. 

Obviously handling all parts of a data structure like it could be missing would not make sense. But a strategy that does not aggressively fail on missing data, will make the client more resilient. 

Also, if during implementation situations where it seems reasonable that data could be missing are identified, but the specifications does not say anything about how that is handled, please raise a Issue for this Change Request describing the situation, so that a future version of the specification can deal with the situation explicitly. 

### Changes in cardinality ### 
Related to additional and missing data, one of the more frequent corrections to the specification is that the cardinality of data changes. E.g. it may be realized that element PreviousStop is specified as 1:1, but when starting the route there is no previous stop, so it should have been 0:1. If the client can tolerate some such changes, it is more likely to be compatible with future specifications revisions. 

Obviously, such tolerance should only be added/present when it does not add code complexity. If it does add complexity, then the possibility of potential future compatibility is not worth it. 

## Multicast ##
The GNSSLocation, FMStoIP, and VEHICLEtoIP service uses Multicast to distribute data.

### Check origin ip-address of data ###
While this normally should not happen, it is possible to have multiple instances of the same multicast service on the local network. If this happens the client will be subscribed to data from both services which could be very confusing and cause unexpected behavior/data. 

The client should check the origin ip-address of received multicast data and see that this address matches the (DNS-SD) service that the client wanted to use. As a minimum the client should check if data is received from multiple addresses (modules) and log that as an error.  

## Other topic ## 

### Service may restart unexpectedly ###

Very little software is truly perfect; unexpected restarts do happen. The client should have strategy to monitor if it “loses connection” with the service and attempt a re-initialization of the service. If the service becomes unavailable when the client expects it to be available, this should be logged as an error.

### Relying on optional features ### 
Some features of the services are specified as optional. If the client uses/relies on an optional feature, then it should have a strategy for handling a service implementation where this feature is not available.


