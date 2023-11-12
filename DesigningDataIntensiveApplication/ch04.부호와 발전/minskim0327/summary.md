## Formats for Encoding Data

*Backward Compatibility*: Newer code can read data that was written by older code.

*Forward Compatibility*: Older code can read data that was written by newer code.

Backward compatibility is normally not hard to achieve: as author of the newer code, you know the format of data written by older code, and so you can explicitly handle it (if necessary by simply keeping the old code to read the old data). Forward compatibility can be trickier, because it requires older code to ignore additions made by a newer version of the code.

### Language-Specific Formats

Many programming languages come with built-in support for encoding in-memory objects into byte sequences.

The encoding is often tied to a particular programming language, and reading the data in another language is very difficult.

In order to restore data in the same object types, the decoding process needs to be able to instantiate arbitrary classes.

### JSON, XML, and Binary Variants

Moving to standardized encodings that can be written and read by many programming languages, JSON and XML are the obvious contenders.

### Thrift and Protocol Buffers

Thrift IDL looks like this.

```thrift
struct Person {
  1: required string       userName,
  2: optional i64          favoriteNumber,
  3: optional list<string> interests
}
```
An equivalent schema definition for Protocol Buffers looks very similar
```proto
message Person {
    required string user_name       = 1;
    optional int64  favorite_number = 2;
    repeated string interests       = 3;
}
```
### Avro

An Avro IDL might look like this.
```
record Person {
    string               userName;
    union { null, long } favoriteNumber = null;
    array<string>        interests;
}
```

### The Merits of Schemas
As we saw, Protocol Buffers, Thrift, and Avro all use a schema to describe a binary encoding format.

They can be much more compact than the various “binary JSON” variants, since they can omit field names from the encoded data.

The schema is a valuable form of documentation, and because the schema is required for decoding, you can be sure that it is up to date (whereas manually maintained documentation may easily diverge from reality).


## Modes of Dataflow
There are many ways data can flow from one process to another, via:

- `databases`
- `service calls`
- `asynchronous message passing`

### Dataflow Through Databases
In a database, the process that writes to the database encodes the data, and the process that reads from the database decodes it. There may just be a single process accessing the database, in which case the reader is simply a later version of the same process—in that case you can think of storing something in the database as sending a message to your future self.

Backward compatibility is clearly necessary here; otherwise your future self won’t be able to decode what you previously wrote.

### Dataflow Through Services: REST and RPC

#### REST
When HTTP is used as the underlying protocol for talking to the service, it is called a web service.

#### RPC
The RPC model tries to make a request to a remote network service look the same as calling a function or method in your programming language, within the same process (this abstraction is called location transparency).

When an older version of the application updates data previously written by a newer version of the application, data may be lost if you’re not careful.

### Messasge-Passing Dataflow
Asynchronous message-passing systems are somewhere between RPC and databases. This goes via an intermediary called a message broker (also called a message queue or message-oriented middleware), which stores the message temporarily.

Using a message broker has several advantages compared to direct RPC: It can act as a buffer if the recipient is unavailable or overloaded, and thus improve system reliability. It can automatically redeliver messages to a process that has crashed, and thus prevent messages from being lost. It avoids the sender needing to know the IP address and port number of the recipient (which is particularly useful in a cloud deployment where virtual machines often come and go). It allows one message to be sent to several recipients.

