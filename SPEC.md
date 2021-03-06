# Specification of Hummel Protocol

Hummel provides a way to synchronise large files from a server onway to many clients. It's based on a Master-Slave architecture similar to HadoopFS using a master for metadata and slaves to store data.

## Summary 
A directory with data files exists in reach of the master server, which builds a repository of these files. The master server reads meta data and creates compressed archives of all files, stores the meta data and publishes the compressed files to all slaves to replicate. It keeps track of replicated files on the slaves. 

A Client may request metadata of a subtree of the indexed repository directory and receives a list of slaves available for datatransfer.
The client initializes the filetransfer using a selected available slave and verifies using previously received metadata from the master node.

## Communicationflow

### Indexing

`Client ---> Master`: Starting indexing defined directory, build metadata and compressed files

`Master ---> Slave`: Publish hashlist, tell to replicate

`Slave ---> Master`: (On every change on slave) Publish complete hashlist stored on slave (might be not complete, e.g. restricted quota)


### One-Way Sync

`Client ---> Master`: Request index of specified repository directory, return metadata list

`Client ---> Master`: Send list of hashes, request list of Slaveserver per Hash (Use enumeration instead of hashes to save bandwidth?)

`Client ---> Slave`: Get [hash].zip

`Client ---> Master`: Send list of hashes (master keeps track of clients  hashes for statistical usage)

### New Slave

`Slave ---> Master`: Publish hashlist stored on slave (might be empty)

`Master ---> Slave`: Publish hashlist, tell to replicate

### Slave Master Heartbeat

`Slave ---> Master`: Send heartbeat after defined interval

## API

### Masterserver

#### Start build

POST http://master/build

Headers: 
- X-HUMMEL-TOKEN: Writetoken

#### Slave Introduction / Announcement

A slave sends a list of stored hashes of his files. Also used to introduce a new slave with empty hashlist.

`POST http://master/slave/hashlist`

Headers:
- X-HUMMEL-TOKEN: Slavetoken
- X-HUMMEL-SLAVE: http://slave:9000

Request-Body: 
```json
[
  "5a3ade42a900439691ebc22013660cae",
  "b493c2e1813f49fb0ef0d9c1388f97c0",
  "4d64f04401ff74fd62f91b9300075c64"
]
```

Response Codes:
- 200: Ack
- 401: Unauthorized

#### Slave fetches hashlist of master

A slave requests a list of all hashes of the repository.

`GET http://master/slave/hashlist`

Headers:
- X-HUMMEL-TOKEN: Slavetoken
- X-HUMMEL-SLAVE: http://slave:9000

Response-Body:
```json
{
  "slaves": [
    "http://slave1",
    "http://slave2",
    "http://slave3"
  ],
  "hashlist": [
    "5a3ade42a900439691ebc22013660cae",
    "b493c2e1813f49fb0ef0d9c1388f97c0",
    "4d64f04401ff74fd62f91b9300075c64"
  ]
}
```


## Problems

* Repository could not be managed via FTP because of large raw data transmission (maybe server to server works?)






# Plan
- Server indexiert alles (Yoma way)
- Erstellt [hash].gz Dateien

- Client holt Metadata
- Client laedt 
--> Problem: Wie kann man dann zusammenarbeiten?
---> Loesung: Webfrontend um Indexierung anzustossen, FTP zum Zugangteilen