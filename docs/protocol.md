# Protocol

<status>STATUS: early draft</status>

TBD: Server-Client feature negotation,
    includes nonce.

TBD: signed messages from client, as well as unsigned ones, rather than AUTH.

## Client messages

### 0x1 - Query

This is a query for records.

Includes an query id for matching up server replies.

Includes a filter specifying which records are sought.

Servers are expected to reply with:
    * 0x80 Record
    * 0x81 Query Complete
    * 0x82 Query Closed

### 0x2 - Close Query

This is a request to close a query.

Includes an identifier for matching up server replies.

Servers are expected to reply with:
    * 0x82 Query Closed

### 0x3 - Submission

This is the submission of a record.

Includes the record submitted.

Servers are expected to reply with:
    * 0x83 Submission Result

### 0x10 - Sync Init

This is the initialization of a negentropy sync of records

### 0x11 - Sync Data

This is a data packet within a negentropy sync of records

### 0x12 - Sync Close

This is the closing of a negentropy sync of records

## Server messages

### 0x80 - Record

This is a record.

Includes a query id that the record came in on.

### 0x81 - Query Complete

This indicates that a query is complete.  This does not mean the query will close, as
subsequently received records that match the query will be subsequently returned.

Includes a query id that the record came in on.

### 0x82 - Query Closed

This indicates that a query has been closed

It must include a coded reason.

### 0x83 - Submission Result

This returns the result of a submission.

### 0x91 - Sync Data

This is a data packet within a negentropy sync of records

### 0x93 - Sync Error

This is an indication that negentropy sync has failed
