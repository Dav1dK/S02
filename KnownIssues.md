# S02 Know Issues list #

This document contains know issues for the ITxPT 2.1.1 S02 specifications. It contains items that someone implementing towards the relevant part of the specification could potentially benefit of being aware of. Items that falls outside of this, e.g. requests for an additional feature are not part of this document. 

The expectation is that these issues will be addressed in a future revision of the specification. There is no guarantee that that future specification revision will be in line with implementation advice below; implementation advice is a best-effort attempt to outline something that will cause the least amount of problems and/or are most likely to be in line with a future specification update. 

The implementation advice may change without notice! 

This document contains *Issues* in the specification. For general advice see documents in the Guidelines folder.

# S02 Known Issues #

## S02 P00 Networks and Protocols ##

### Section 1.3 “IP Networks” ###

**Issue:** Section 1.3.2 Onboard Passenger IP Network (optional) states that “… shall be configured with relevant and up-to-date security protocols recommended by security experts …”. Which could imply that this is not important/required for modules that are connected to the on-board network.  

**Implementation advice:** All modules, regardless of where they are connected should have up-to-date security. 

### Section 2.1.2.2 “TXT-record” ###

**Issue:** Unlike SRV-record TTL, TXT-record TTL is not stated. 

**Implementation advice:** Use the value recommended in RFC-6762, 75 min (4500s). Note that prior to version 2.1.1 3600s was used for both SRV and TXT record TTL in all examples. In the 2.1.1 specification this have been changed to the values recommended by RFC-6762. 

## S02 P01 Inventory ##

### Section 3.3 “TXT record” ###

**Issue:** In Note 4 it is stated that “Modules type declared with reference to specified ITxPT service shall implement related ITxPT service in addition to mandatory module inventory service. For example, “MADT” type shall implement MADT service and module inventory service…”. This is not in line with what part S01 of the spec says.  

**Implementation advice:** Before the release of the next minor/major specification, a decision needs to be taken if Module type is coupled to services, or not. For now, ITxPT will accept that the service is not present, and only evaluate it based on the description in S01 Module types table.  

## S02 P04 “FMStoIP” ## 

**Issue:** There are a number of known issues in the 2.1.1 FMStoIP specification. Technical WorkGroup 01 Maintenance – SG1 FMStoIP is currently (Q4 2020) working on addressing these issues and expects a draft update to be issued Q2 2021.  

**Implementation advice (service):** The recommendation would be to skip the 2.1.0/2.1.1 FMStoIP specification, and go directly to 2.2.0

**Implementation advice (clients):** The recommendation would be to skip the 2.1.0/2.1.1 FMStoIP specification, and go directly to 2.2.0. If a client that also works with 2.1.0/2.1.1 FMStoIP service is wanted, it should be trivially easy to do that.

## S02 P09 “MQTT” ##

### Section 3.3 TXT record ###

**Issue:** It is not stated explicitly what the txt-record attribute ‘topic’ points to.  

**Implementation advice:** The intention is that the path given by `topic=<path>` is the itxpt root-topic. Directly under `<path>` one will find all the top-level topics specified by ITxPT MQTT service specifications. This implies firstly that there should not be *any* non-itxpt topics under `<path>` as those may conflict with topics specified (in the future) by ITxPT, and secondly that there should not – currently – be any topics of `<path>` as the 2.1.1 specification does not contain any ITxPT MQTT services.  