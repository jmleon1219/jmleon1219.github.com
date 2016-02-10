---
layout: post
title:  "The Data Vault Alternative"
date:   2016-02-08 00:00:00 
categories: Data Vault Data Warehousing
---
There are a few common techniques for persisting data. A Kimball approach is characterized by many subject based dimensions feeding into fact table where all the measurements reside.  Once a critical mass of dimensions are created they can be re-used and the only development time required is to create the fact tables this is referred to as the "Data Warehouse Bus".  An Inmon approach is to create a 3rd normal form design at the atomic level.  Also known as the "Corporate Information Factory", this approach focuses on creating a fully formed data warehouse then generating the reporting marts from the data warehouse.  Finally there is the Data Vault.

## The Data Vault
The data vault is commonly described as an approach that takes the best parts of both the Kimball and Inmon.  At the core of the data vault is the Link-Hub-Satellite relationship.  Each hub represents a business key in a source system, examples include customer, employee and purchase order.  One of the key concepts is that hubs cannot be in direct contact with each other.  

A link has to exist to connect the two hubs.  The link is always a many to many relationship that uses the surrogate key from each hub.  The many to many relationship in the link needs to exist even if the actual relationship is a 1:M.  The N:M relationship gives the model the flexibility to accommodate any special circumstances that might arise in the future.

Finally, the satellites are needed to hold the hub's attribute data.  The satellites main function is to describe the hub, thus they do not need a surrogate key of their own.  An individual hub can be referenced by many individual satellites.  How often the data changes, the source system and the time at which the data was added impacts the number of satellites needed for a hub.  Each row is timestamped with a start and end to be able to re-create an entire record at a certain point in time. 

## Applicability
Being a veteran of both the Inmon and Kimball methods to organize information, the Data Vault stood out in two major respects.  The ability to audit entire solution to the line level and the flexibility to accommodate changes that were not in the initial design.

From a Kimball perspective, the most pain comes when there needs to be a change to an existing dimension.  To make the dimension conform to numerous stars the data occasionally needs to be snow flaked or the grain needs to be altered.  Such changes cause ripple effects throughout the entire model, ETL layer and reporting layer.  If there are new fields added to the dimension, the audit-ability of the dimension comes into question.

The effort to create a data vault is not trivial.  Similar to the Inmon approach there still is a need to spin off reporting marts off of the data vault.  This does not exactly double the workload considering that the building of the data vault will considerably simplify the building of each data mart.  A work effort of 1.5 the duration to build a Kimball star schema seems fair.  

##Is such a sophisticated solution necessary?
The most compelling scenario for a data vault is if reporting across business functions is a necessity.  A data vault could manage the relationships between hubs in different business units.  Another scenario is if data auditing is a paramount concern.  A data vault can create new rows from systems that currently overwrite exiting values.  Conversely, if the organization is only concerned with creating business unit specific insights then a Kimball style approach will suffice.  