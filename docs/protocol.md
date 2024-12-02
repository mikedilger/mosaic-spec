# Protocol

<status>PAGE STATUS: incomplete</status>

Protocol message are not separately digitally signed due to the TLS
transport and certificate authentication.

## Feature Negotiation

Protocol feature negotiation is done on a transport-by-transport level.
For [WebSockets](websockets.md) transport (the default) this is done with
the `X-Mosaic-Features` header.

No features have been defined yet.

## Messages

Every message starts with a one-byte type, shown below in the header
of each type. Following this is the data of the message.

## Client messages

### Query

> **0x1**

This is a query for records in the following format:

```text
                    1       2       3
    0       8       6       4       2
 0  +-------------------------------+
    |  0x1  |   QUERY_ID    | 0x0   |
    +-------------------------------+
    |  FILTER LEN   | 0x0           |
    +-------------------------------+
    |  FILTER ...                   |
    |  ...                          |
    +-------------------------------+
```

The `QUERY_ID` should be made up by the client. It can be any 16-bit
number and is used for associating responses.

`FILTER_LEN` is a 16-bit little-endian integer indicating the length
of the filter.

The `FILTER` is defined in [filter](filter.md).

This is a client initiated message. Servers are expected to reply with:

* a series of zero or more [`Record`](#record) messages representing all
  the matching records on the server initially, followed by
  a [`Query Complete`](#query-complete) message, potentially followed by zero or
  more [`Record`](#record) messages that flow in to the server after
  the initial response (so long as the query is still open), or
* a [`Query Closed`](#query-closed) message if the query could not be served.

### Close Query

> **0x2**

This is a client request to close a query in the following format:

```text
                    1       2       3
    0       8       6       4       2
 0  +-------------------------------+
    |  0x2  |   QUERY_ID    | 0x0   |
    +-------------------------------+
```

This is a client initiated message. Servers are expected to reply with:

* [`Query Closed`](#query-closed)

### Submission

> **0x3**

This is the submission of a record in the following format:

```text
                    1       2       3
    0       8       6       4       2
 0  +-------------------------------+
    |  0x3  |   LENGTH              |
    +-------------------------------+
    | RECORD ...                    |
    | ...                           | 
    +-------------------------------+
```

`LENGTH` is a 24-bit little-endian length, with a maximum value of the max length
of a record (1024576 bytes) and representing the actual length of the subsequent
record submitted.

`RECORD` is the record.

This is a client initiated message. Servers are expected to reply with:

* [`Submission Result`](#submission-result) with a hash prefix matching the record.

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

### Record

> **0x80**
This is a record.

Includes a query id that the record came in on.

### Query Complete

> **0x81**

This indicates that a query is complete.  This does not mean the query will
close, as subsequently received records that match the query will be
subsequently returned.

Includes a query id that the record came in on.

### Query Closed

> **0x82**

This indicates that a query has been closed

It must include a coded reason.

### Submission Result

> **0x83**

This returns the result of a submission.

### Sync Data

> **0x91**

This is a data packet within a negentropy sync of records

### Sync Error

> **0x93**

This is an indication that negentropy sync has failed
