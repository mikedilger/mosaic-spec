# Protocol

<status>PAGE STATUS: early draft</status>

Protocol message are not separately digitally signed due to the TLS
transport and certificate authentication.

## Protocol Extensions

Protocol extension negotiation is done on a transport-by-transport level.
For [WebSockets](websockets.md) transport (the default) this is done with
the `X-Mosaic-Extensions` header.

The following extensions have been defined:

* [Sync Protocol Extension](sync_protocol_extension.md)

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

`LENGTH` is a 24-bit little-endian length, with a maximum value of the max
length of a record (1024576 bytes) and representing the actual length of the
subsequent record submitted.

`RECORD` is the record.

This is a client initiated message. Servers are expected to reply with:

* [`Submission Result`](#submission-result) with a hash prefix matching the record.

## Server messages

### Record

> **0x80**

This is a record returned from a query in the following format:

```text
                    1       2       3
    0       8       6       4       2
 0  +-------------------------------+
    |  0x80 |   QUERY_ID    | 0x0   |
    +-------------------------------+
    |  0x0  |   LENGTH              |
    +-------------------------------+
    | RECORD ...                    |
    | ...                           | 
    +-------------------------------+
```

The `QUERY_ID` is the client query that this record matched.

The `LENGTH` is the length of the record as a 24-bit little-endian
encoded value.

The `RECORD` is the actual record

This is a server response message in response to [`Query`](#query).

### Query Complete

> **0x81**

This is a message indicating that a query is complete, in the following
format:

```text
                    1       2       3
    0       8       6       4       2
 0  +-------------------------------+
    |  0x81 |   QUERY_ID    | 0x0   |
    +-------------------------------+
```

This indicates that a query is complete. This does not mean the query will
close, as subsequently received records that match the query will be
subsequently returned.

This is a server response message in response to [`Query`](#query).

### Query Closed

> **0x82**

This is a message indicating that a query has been closed, in the following
format:

```text
                    1       2       3
    0       8       6       4       2
 0  +-------------------------------+
    |  0x82 |   QUERY_ID    | CODE  |
    +-------------------------------+
```

The `QUERY_ID` is the client query that is being closed that was
previously supplied by the client in a [`Query`](#query).

The `CODE` indicates the reason for closure, from among the following
defined reasons.

* `ON_REQUEST`: 0x1 - In response to [`Close`](#close)
* `REJECTED_INVALID`: 0x10 - Query was rejected due to being invalid
* `REJECTED_TOO_OPEN`: 0x11 - Query was rejected due to being too open
   (scraping too many records)
* `REJECTED_TOO_FAST`: 0x12 - Query was rejected due to too many queries
   (or messages of any type) being submitted recently by this client
* `REJECTED_TEMP_BANNED`: 0x13 - Query was rejected due to the client
   beint temporarily banned
* `REJECTED_PERM_BANNED`: 0x14 - Query was rejected due to the client
   beint permanently banned
* `SHUTTING_DOWN`: 0x30 - The server is shutting down
* `INTERNAL_ERROR`: 0xF0 - A server error occured
* `OTHER`: 0xFF - Some other reason

### Submission Result

> **0x83**

This is a message returning the result of a [Submission](#submission) in
the following format:

```text
                    1       2       3
    0       8       6       4       2
 0  +-------------------------------+
    |  0x82 |  0x0  |  0x0  |  CODE |
    +-------------------------------+
    |  HASH PREFIX 1/8              |
    +-------------------------------+
    |  HASH PREFIX 2/8              |
    +-------------------------------+
    |  HASH PREFIX 3/8              |
    +-------------------------------+
    |  HASH PREFIX 4/8              |
    +-------------------------------+
    |  HASH PREFIX 5/8              |
    +-------------------------------+
    |  HASH PREFIX 6/8              |
    +-------------------------------+
    |  HASH PREFIX 7/8              |
    +-------------------------------+
    |  HASH PREFIX 8/8              |
    +-------------------------------+
```

The `CODE` indicates the result of the submission from among the following
defined results:

* `OK`: 0x1 - Record submission was accepted
* `DUPLICATE`: 0x2 - Record is a duplicate. Servers may use this or
  they may optionally use `OK` in the same circumstance.
* `REJECTED_INVALID`: 0x10 - Record is invalid
* `REJECTED_TOO_FAST`: 0x12 - Record submission was rejected due to too many
   submissions (or messages of any type) being made recently by this client
* `REJECTED_TEMP_BANNED`: 0x13 - Record submission was rejected due to the
   client being temporarily banned
* `REJECTED_PERM_BANNED`: 0x14 - Record submission was rejected due to the
   client being permanently banned
* `REJECTED_REQUIRES_AUTHN`: 0x15 - Record submission requires authentication
   but the client is connected anonymously.
* `REJECTED_REQUIRES_AUTHZ`: 0x16 - Record submission requires authorization
   (e.g. an account with the server) which the client user does not have.
* `INTERNAL_ERROR`: 0xF0 - A server error occured
* `OTHER`: 0xFF - Some other reason

FIXME: add proof of work (not defined yet)

The `HASH_PREFIX` is the first 256-bits of the event hash.