# S02 - Onboard Architecture specifications #

This it the github repository for the ITxPT S02 specifications. It contains additional files that complement the specifications. 

The specifications themselves are found at https://wiki.itxpt.org/index.php?title=ITxPT_Technical_Specifications

The **branch name** corresponds to the **specification revision** these files are valid for.

## Know Issues List ##

Known Issues for the S02 Specifications are documented in [KnownIssues.md](KnownIssues.md)

## Development Guides ##

Find implementation advice for Clients in [ClientDevelopmentGuide.md](Guidelines/ClientDevelopmentGuide.md)

FMStoIP specific implementation advice for both Service and Clients [FMStoIP Implementation Guide](Guidelines/fmstoip_impl_guide.md)

## Schemas ## 

Schema files and examples are currently available for each of the following services:
- ModuleInventory
- GNSSLocation
- FMStoIP
- VEHICLEtoIP
- AVMS
- APC
- MADT

### Purpose of schema files ###
XML Schema Definition (XSD) or JSON Schema are the standard for describing XML and JSON structured data. Schemas should be used as a software development tool in the implementation of ITxPT IP service using XML or JSON payloads. It can validate that a data structure is well formed and compliant with the structure defined in the related ITxPT specification. In some cases, Schema can be used to automate/simplify the generation of payload structure at coding level.

### Report an issue/question related to the schema files ###
To report any issue found during implementation or question you have regarding the content of this repository, please use the native GitHub Issue system. 

## Report an issue/question related to an ITxPT specification ###
To report any issue found during implementation or question you have regarding the content of the specification, please use the dedicated form: https://forms.gle/tMRgxTXSF1Mahpq27
