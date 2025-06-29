# Mosaic Sync Protocol Extension

<status>PAGE STATUS: incomplete</status>

This extends the protocol with negentropy support.

The following messages are added:

## Client messages

### Sync Init

> **0x10**

This is the initialization of a negentropy sync of records

### Sync Data

> **0x11**

This is a data packet within a negentropy sync of records

### Sync Close

> **0x12**

This is the closing of a negentropy sync of records

## Server messages

### Sync Data

> **0x91**

This is a data packet within a negentropy sync of records

### Sync Error

> **0x93**

This is an indication that negentropy sync has failed
