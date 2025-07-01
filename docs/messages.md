# Protocol messages

<status>PAGE STATUS: early draft</status>

Protocol message are not separately digitally signed due to the TLS
transport and certificate authentication.

All protocol messages start with the same 4 byte sequence consisting
of one byte for the type, and three bytes for the full byte length of
the entire protocol message encoded in little-endian format. This
allows the protocol to be easily framed within a stream.

## Protocol Extensions

Protocol extension negotiation is done on a transport-by-transport level.
For [QUIC](quic.md) transport this is done with the initial packet.
For [WebSockets](websockets.md) transport this is done with the
`X-Mosaic-Extensions` header.

The following extensions have been defined:

* [Sync](sync.md)

## Asynchronous

All client messages initiate an action, and all server messages are in
response to client messages (NOTE: this may not apply to Mosaic
extensions). Yet messaging is asychronous and both sides SHOULD be
prepared to deal with a message sent at any time.

Servers SHOULD not send messages until they receive a message that
requires a response.

If a server fails to receive a message from a client that does not
have an open query or submission within a reasonable timeframe, it
MAY close the connection.

If a client receives a server message that was not expected, it MUST
ignore it and decide what to do next. In some cases, clients MAY close
the connection because the service they were seeking failed. In other
cases they MAY try something else. There is no provision for a client
to alert a server of an error.

## Messages

Every message starts with a one-byte type, shown below in the header
of each type. Following this is the data of the message.

| Initiator | Message | Type |
|-----------|---------|------|
| Client | [Get](#get) | 0x1 |
| Client | [Query](#query) | 0x2 |
| Client | [Subscribe](#subscribe) | 0x3 |
| Client | [Unsubscribe](#unsubscribe) | 0x4 |
| Client | [Submission](#submission) | 0x5 |
| Server | [Record](#record) | 0x80 |
| Server | [Locally Complete](#locally-complete) | 0x81 |
| Server | [Query Closed](#query-closed) | 0x82 |
| Server | [Submission Result](#submission-result) | 0x83 |
| Either | [Unrecognized](#unrecognized) | 0xF0 |

---

## Client Messages

### Get

> **0x1**

This is a query for specific records.

It has the following format:


```text

    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0x1 |     LENGTH      | QUERY_ID  |   0x0     |
 8  +-----------------------------------------------+
    | Mixed IDs or ADDRs, each one 48 bytes long... |
    | ...                                           |
    +-----------------------------------------------+
```

* `[0:1]` - The type 0x1
* `[1:4]` - The byte length of this message, in little-endian format
* `[4:6]` - `QUERY_ID`, two bytes which should be made up by the client and
  used to associate returned [`Record`](#record) responses to this request.
* `[6:8]` - Zeroed
* `[*]` - A sequence of mixed IDs and ADDRs.  Note that ADDRs start
  with a 1 bit, whereas IDs start with a 0 bit, and both of them are 48
  bytes long.

This is a client initiated message. Servers are expected to reply with:

* A series of zero or more [`Record`](#record) messages representing all
  the matched records on the server, followed by a
  [`Query Closed`](#query-closed) message.

### Query

> **0x2**

This is a query for records that closes once served.

It has the following format:

```text
    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0x2 |     LENGTH      | QUERY_ID  |  LIMIT    |
 8  +-----------------------------------------------+
    | FILTER ...                                    |
	| ...                                           |
    +-----------------------------------------------+
```

* `[0:1]` - The type 0x2
* `[1:4]` - The byte length of this message, in little-endian format
* `[4:6]` - `QUERY_ID` two bytes which should be made up by the client and used
  to associate returned [`Record`](#record) responses to this request.
* `[6:8]` - `LIMIT`, an unsigned integer in little-endian format, specifies
  the maximum number of responses that the client wishes to receive.  A value
  of 0 indicates unlimited.
* `[*]` - The `FILTER`, see [Filter](filter.md).

This is a client initiated message. Servers are expected to reply with:

* a series of zero or more [`Record`](#record) messages representing all
  the matching records on the server initially, followed by
  a [`Query Closed`](#query-closed) message.

Queries MUST return results in anti-chronological order, from most
recent backwards.

### Subscribe

> **0x3**

This is a query for records that is kept open after the initial records are
served so that newly arriving records that match can be sent to the client
immediately.

It has the following format:

```text
    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0x3 |     LENGTH      | QUERY_ID  |   LIMIT   |
 8  +-----------------------------------------------+
    | FILTER ...                                    |
	| ...                                           |
    +-----------------------------------------------+
```

* `[0:1]` - The type 0x3
* `[1:4]` - The byte length of this message, in little-endian format
* `[4:6]` - `QUERY_ID`, two bytes which should be made up by the client and
  used to associate returned [`Record`](#record) responses to this request.
* `[6:8]` - `LIMIT`, an unsigned integer in little-endian format, specifies
  the maximum number of responses that the client wishes to receive.  A value
  of 0 indicates unlimited.
* `[*]` - The `FILTER`, see [Filter](filter.md).

This is a client initiated message. Servers are expected to reply with:

* a series of zero or more [`Record`](#record) messages representing all
  the matching records on the server initially, followed by
  a [`Locally Complete`](#locally-complete) message, potentially followed by
  zero or
  more [`Record`](#record) messages that flow in to the server after
  the initial response (so long as the query is still open), or
* a [`Query Closed`](#query-closed) message if the query could not be served.

Queries MUST return results in anti-chronological order, from most
recent backwards.

### Unsubscribe

> **0x4**

This is a client request to close an open subscription query.

It has the following format:

```text
    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0x4 |     LENGTH      | QUERY_ID  |   0x0     |
 8  +-----------------------------------------------+
```

* `[0:1]` - The type 0x3
* `[1:4]` - The byte length of this message, in little-endian format
* `[4:6]` - `QUERY_ID`, two bytes indicating which query should be closed.
* `[4:8]` - Zeroed

This is a client initiated message. Servers are expected to reply with:

* [`Query Closed`](#query-closed)

### Submission

> **0x5**

This is the submission of a record.

It has the following format:

```text
    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0x5 |    LENGTH       |          0x0          |
 8  +-----------------------------------------------+
    | RECORD ...                                    |
	| ...                                           |
    +-----------------------------------------------+
```

* `[0:1]` - The type 0x5
* `[1:4]` - The byte length of this message, in little-endian format
* `[4:8]` - Zeroed
* `[*]` - `RECORD` is the record submitted

This is a client initiated message. Servers are expected to reply with:

* [`Submission Result`](#submission-result) with an id prefix matching the record.

---

## Server Messages

### Record

> **0x80**

This is a record returned from a query.

It has the following format:

```text
    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0x80|     LENGTH      | QUERY_ID  |   0x0     |
 8  +-----------------------------------------------+
    | RECORD ...                                    |
	| ...                                           |
    +-----------------------------------------------+
```

* `[0:1]` - The type 0x80
* `[1:4]` - The byte length of this message, in little-endian format
* `[4:6]` - `QUERY_ID` indicates the client query that this record matched.
* `[6:8]` - Zeroed
* `[*]` - `RECORD` is the returned record.

This is a server response message in response to [`Get`](#get)
or [`Query`](#query) or [`Subscribe`](#subscribe).

### Locally Complete

> **0x81**

This is a message indicating that a query has served all matching local records,
but remains open to serve matching records that subsequently arrive.

It has the following format:

```text
    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0x81|     LENGTH      |  QUERY_ID |    0x0    |
 8  +-----------------------------------------------+
```

* `[0:1]` - The type 0x81
* `[1:4]` - The byte length of this message, in little-endian format
* `[4:6]` - `QUERY_ID` indicates the client query that is now locally complete.
* `[6:8]` - zeroed

This is a server response message in response to [`Subscribe`](#subscribe).

### Query Closed

> **0x82**

This is a message indicating that a query has been closed.

It has the following format:

```text
    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0x82|     LENGTH      |  QUERY_ID | CODE| 0x0 |
 8  +-----------------------------------------------+
```

* `[0:1]` - The type 0x82
* `[1:4]` - The byte length of this message, in little-endian format
* `[4:6]` - `QUERY_ID` indicates the client query that is now closed.
* `[6:7]` - `CODE` indicates the reason for closure from among the
  following defined reasons:
    * `ON_REQUEST`: 0x1 - Query was closed in response to an [`Unsubscribe`](#unsubscribe) request
    * `REJECTED_INVALID`: 0x10 - Query is invalid
    * `REJECTED_TOO_OPEN`: 0x11 - Query is too broad, matching too many records
    * `REJECTED_TOO_FAST`: 0x12 - Queries (or messages) are coming too quickly. Slow down
    * `REJECTED_TEMP_BANNED`: 0x13 - Client is temporarily banned from querying
    * `REJECTED_PERM_BANNED`: 0x14 - Client is permanently banned from querying
    * `SHUTTING_DOWN`: 0x30 - The server is shutting down
    * `INTERNAL_ERROR`: 0xF0 - The server has encountered an internal error
    * `OTHER`: 0xFF - Other reason, or not specified
* `[7:8]` - zero



### Submission Result

> **0x83**

This is a message returning the result of a [Submission](#submission).

It has the following format:

```text
    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0x83|     LENGTH      |CODE |      0x0        |
 8  +-----------------------------------------------+
    | ID PREFIX bytes 0..8                          |
16  +-----------------------------------------------+
    | ID PREFIX bytes 8..16                         |
24  +-----------------------------------------------+
    | ID PREFIX bytes 16..24                        |
32  +-----------------------------------------------+
    | ID PREFIX bytes 24..32                        |
40  +-----------------------------------------------+
```

* `[0:1]` - The type 0x83
* `[1:4]` - The byte length of this message, in little-endian format
* `[4:5]` - `CODE` which indicates the result of the submission from among the
  following defined results:
    * `OK`: 0x1 - Record submission was accepted
    * `DUPLICATE`: 0x2 - Record is a duplicate. Servers may use this or
      they may optionally use `OK` in the same circumstance.
    * `NO_CONSUMERS`: 0x3 - Ephemeral record had no consumers. Servers may use this or
      they may optionally use `OK` in the same circumstance.
    * `REJECTED_INVALID`: 0x10 - Record is invalid
    * `REJECTED_TOO_FAST`: 0x12 - Submissions (or messages) are coming too quickly. Slow down.
    * `REJECTED_TEMP_BANNED`: 0x13 - Client is temporarily banned from submisisons.
    * `REJECTED_PERM_BANNED`: 0x14 - Client is permanently banned from submissions.
    * `REJECTED_REQUIRES_AUTHN`: 0x15 - Submission requires authentication.
    * `REJECTED_REQUIRES_AUTHZ`: 0x16 - Submission requires authorization.
    * `INTERNAL_ERROR`: 0xF0 - The server has encountered an internal error
    * `OTHER`: 0xFF - Other reason, or not specified
* `[5:8]` - Zeroed
* `[8:40]` - A 32-byte prefix of the 48-byte Id.

### Unrecognized

> **0xF0**

This is a message indicating that the last message from the peer was not recognized.

When this is received over QUIC transport, the stream should be closed and further messaging
can occur on other streams.

It has the following format:

```text
    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0xF0|     LENGTH      |         0x0           |
 8  +-----------------------------------------------+
```

* `[0:1]` - The type 0xF0
* `[1:4]` - The byte length of this message, in little-endian format
* `[4:8]` - Zeroed
