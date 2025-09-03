# Protocol messages

Protocol message are not separately digitally signed due to the TLS
transport and certificate authentication.

All protocol messages start with the same 4 byte sequence consisting
of one byte for the type, and three bytes for the full byte length of
the entire protocol message encoded in little-endian format. This
allows the protocol to be easily framed within a stream.

## Asynchronous

All client messages initiate an action, and all server messages are in
response to client messages (NOTE: this MAY not apply to Mosaic
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
| Client | [Hello](#hello) | 0x10 |
| Server | [Hello Ack](#hello-ack) | 0x90 |
| Client | [Hello Auth](#hello-auth) | 0x11 |
| Server | [Closing](#closing) | 0xFE |
|        |            |     |
| Client | [Get](#get) | 0x1 |
| Client | [Query](#query) | 0x2 |
| Client | [Subscribe](#subscribe) | 0x3 |
| Server | [Record](#record) | 0x80 |
| Server | [Locally Complete](#locally-complete) | 0x81 |
| Client | [Unsubscribe](#unsubscribe) | 0x4 |
| Server | [Query Closed](#query-closed) | 0x82 |
|        |            |     |
| Client | [Submission](#submission) | 0x5 |
| Server | [Submission Result](#submission-result) | 0x83 |
|        |            |     |
| Client | [BLOB Get](#blob-get) | 0x8 |
| Server | [BLOB Result](#blob-result) | 0x86 |
| Client | [BLOB Submission](#blob-submission) | 0x7 |
| Server | [BLOB Submission Result](#blob-submission-result) | 0x85 |
|        |            |     |
| Client | [DHT Lookup](#dht-lookup) | 0x6 |
| Server | [DHT Response](#dht-response) | 0x84 |
|        |            |     |
| Either | [Unrecognized](#unrecognized) | 0xF0 |

---

## Client Messages

### Hello

NOTE: [WebSockets](websockets.md) sends HELLO information out of band and does
not utilize this message.

This is the initial message sent from the client to the server. It specifies
the highest Mosaic version that the client supports, as well as the applications
that the client is requesting to utilize.

It has the following format:


```text

    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0x10|     LENGTH      | ZEROED    | MOSAIC_MAJOR_VERSION  |
 8  +-----------------------------------------------+
    | APP_ID                | ...                   |
    +-----------------------------------------------+
```

* `[0:1]` - The type 0x10
* `[1:4]` - The byte length of this message, in little-endian format
* `[4:6]` - Zeroed
* `[6:8]` - `MOSAIC_MAJOR_VERSION` - The highest Mosaic major version number that the client supports, in little-endian format
* `[*]` - A sequence of 32-bit [Application](applications.md) IDs that the client wishes to use, in little-endian format

This is a client initiated message. Servers are expected to reply with [Hello Ack](#hello-ack).

### Hello Auth

Only WebSockets utilizes this message.
[QUIC](quic.md) and [TCP](tcp.md) handle authentication at the TLS layer.

TBD

### Get

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
* `[4:6]` - `QUERY_ID`, two bytes which SHOULD be made up by the client and
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
* `[4:6]` - `QUERY_ID` two bytes which SHOULD be made up by the client and used
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
* `[4:6]` - `QUERY_ID`, two bytes which SHOULD be made up by the client and
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
recent backwards, up until `Locally Complete` is served, after which point
they return records as they come in.

### Unsubscribe

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
* `[4:6]` - `QUERY_ID`, two bytes indicating which query SHOULD be closed.
* `[4:8]` - Zeroed

This is a client initiated message. Servers are expected to reply with:

* [`Query Closed`](#query-closed)

### Submission

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

### BLOB Get

A BLOB is a Binary Large OBject. This message requests retrieval of a BLOB

It has the following format:

```text
    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0x8 |                 ZEROED                  |
 8  +-----------------------------------------------+
    | HASH   ...                                    |
	| ...                                           |
40  +-----------------------------------------------+
```

* `[0:1]` - The type 0x8
* `[1:8]` - Zeroed
* `[8:40]` - the 256-bit BLAKE3 hash of the BLOB.

This is a client initiated message. Servers are expected to reply with:

* [`BLOB Result`](#blob-result) containing the BLOB or an error.

### BLOB Submission

A BLOB is a Binary Large OBject. This message submits a BLOB to the server for storage and later
retrieval.

It has the following format:

```text
    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0x7 |  0  |              LENGTH               |
 8  +-----------------------------------------------+
    | HASH   ...                                    |
	| ...                                           |
40  +-----------------------------------------------+
    | BLOB ...                                      |
    |                                          ...  |
```

* `[0:1]` - The type 0x7
* `[1:2]` - Zero
* `[2:8]` - The length of the binary data in little-endian format.
* `[8:40]` - the 256-bit BLAKE3 hash of the binary data.
* `[40:]` - The binary data

This is a client initiated message. Servers are expected to reply with:

* [`BLOB Submission Result`](#blob-submission-result) with a hash matching the record.

Servers MAY require authentication and MAY remember who submitted which BLOBs. Servers may limit
the size of submitted BLOBs. Servers MAY provide some means for deleting BLOBs no longer in use.
Servers MAY delete BLOBs that violate server policy. These issues are currently out of scope of
this spec.

### DHT Lookup

This requests that the server perform a DHT lookup on behalf of the client.
[<sup>rat</sup>](rationale.md#dht-lookup-by-server)

TBD

---

## Server Messages

### Hello Ack

NOTE: [WebSockets](websockets.md) sends HELLO ACK information out of band and does
not utilize this message.

This is a reply to the client [`Hello`](#hello) message. It includes the highest
Mosaic version that both parties support, as well as all of the application IDs
that the client requested which the server can support.

It has the following format:

```text

    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0x90|     LENGTH      |RESULT|    | MOSAIC_MAJOR_VERSION  |
 8  +-----------------------------------------------+
    | APP_ID                | ...                   |
    +-----------------------------------------------+
```

* `[0:1]` - The type 0x90
* `[1:4]` - The byte length of this message, in little-endian format
* `[4:5]` - `RESULT` indicates the result, typically SUCCESS but
  possibly DUPLICATE. Servers should refrain from using errors or rejections
  in this result and instead if necessary issue a [Closing](#closing) message.
* `[5:6]` - Zeroed
* `[6:8]` - `MOSAIC_MAJOR_VERSION` - The highest Mosaic major version number
  that both the server and the client supports, in little-endian format
* `[*]` - A sequence of 32-bit [Application](applications.md) IDs that the client
  requested and that the server can also support, in little-endian format

### Closing

This is a server message indicating that the server is closing the connection.
It is not necessarily in response to any client message, but could be sent
in response to any client message.

```text
    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0xFE|RESULT|           ZEROED                 |
 8  +-----------------------------------------------+
```

* `[0:1]` - The type 0xFE
* `[1:2]` - Result Code giving the reason. See [Result Codes](#result-codes).
* `[2:8]` - Zeroed

### Record

This is a record returned from a query (`Get`, `Query`, or `Subscribe`).

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

This is a message indicating that a query has been closed.

It has the following format:

```text
    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0x82|     LENGTH      |  QUERY_ID |RESULT| 0x0|
 8  +-----------------------------------------------+
```

* `[0:1]` - The type 0x82
* `[1:4]` - The byte length of this message, in little-endian format
* `[4:6]` - `QUERY_ID` indicates the client query that is now closed.
* `[6:7]` - `RESULT` indicates the reason for closure.
  See [Result Codes](#result-codes).
* `[7:8]` - zero


### Submission Result

This is a message returning the result of a [Submission](#submission).

It has the following format:

```text
    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0x83|     LENGTH      |RESULT|     0x0        |
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
* `[4:5]` - `RESULT` which indicates the result of the submission.
  See [Result Codes](#result-codes).
* `[5:8]` - Zeroed
* `[8:40]` - A 32-byte prefix of the 48-byte Id.

### BLOB Result

This is a server response to a [`BLOB Get`](#blob-get) request returning the BLOB
or an error condition.

```text
    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0x86|RESULT|            LENGTH                |
 8  +-----------------------------------------------+
    | HASH   ...                                    |
	| ...                                           |
40  +-----------------------------------------------+
    | BLOB ...                                      |
    |                                          ...  |
```

* `[0:1]` - The type 0x86\
* `[1:2]` - The result of the blob get request.
  See [Result Codes](#result-codes).
* `[2:8]` - The length of the binary data in little-endian format.
        If the result was not a success, this should be 0.
* `[8:40]` - the 256-bit BLAKE3 hash of the binary data.
* `[40:]` - The binary data if the request was successful.

### BLOB Submission Result

This is a server response to a [`BLOB Submission`](#blob-submission) request.

```text
    0     1     2     3     4     5     6     7     8
 0  +-----------------------------------------------+
    | 0x85|RESULT|           ZEROED                 |
 8  +-----------------------------------------------+
    | HASH   ...                                    |
	| ...                                           |
40  +-----------------------------------------------+
```

* `[0:1]` - The type 0x85
* `[1:2]` - The result. See [Result Codes](#result-codes).
* `[2:8]` - Zeroed
* `[8:40]` - the 256-bit BLAKE3 hash of the binary data.

### DHT Response

This is a server response to a [`DHT Lookup`](#dht-lookup) request.

TBD

### Unrecognized

This is a message indicating that the last message from the peer was not recognized.

When this is received over QUIC transport, the stream SHOULD be closed and further messaging
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

## Result Codes

Multiple protocol messages contain a 1-byte result code. The definition
of these codes is shared between message types and defined here.

* 0 - `UNDEFINED` - This result code is not defined. It should not be used.

Successes:

* 1 - `SUCCESS` - Generic success message
* 2 - `ACCEPTED` - Accepted a submission
* 3 - `DUPLICATE` - A record submitted is a duplicate of an existing record.
* 4 - `NO_CONSUMERS` - Ephemeral record had no consumers.

Failures:

* 16 - `NOT_FOUND` - Record or BLOB was not found

User Errors:

* 32 - `REQUIRES_AUTHENTICATION` - Rejected as the request requires authentication
* 33 - `UNAUTHORIZED` - Rejected as the pubkey is not authorized for the action
* 36 - `INVALID` - Request was invalid
* 37 - `TOO_OPEN` - A Query or Subscribe was too open, potentially
        matching too many records
* 38 - `TOO_LARGE` - The submission was too large
* 39 - `TOO_FAST` - Requests are coming in too fast from this client, or of this type.

User Rejections:

* 48 - `IP_TEMP_BANNED` - IP address is temporarily banned
* 49 - `IP_PERM_BANNED` - IP address is permanently banned
* 50 - `PUBKEY_TEMP_BANNED` - Pubkey is temporarily banned
* 51 - `PUBKEY_PERM_BANNED` - Pubkey is permanently banned

Server errors:

* 64 - `SHUTTING_DOWN` - Server is shutting down
* 65 - `TEMPORARY ERROR` - Temporary server error
* 66 - `PERSISTENT ERROR` - Persistent server error
* 67 - `GENERAL ERROR` - General server error

## Protocol Extensions

Protocol extension negotiation is done on a transport-by-transport level.
For [QUIC](quic.md) transport this is done with the initial packet.
For [WebSockets](websockets.md) transport this is done with the
`X-Mosaic-Extensions` header. See the specific transport.

The following extensions have been defined:

* [Sync](sync.md)

