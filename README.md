# ploppy

Like Dropbox and Owncloud, but better

## Description

ploppy is a peer-to-peer file replication and distribution system which has no centralised database (unlike Dropbox) and works on servers as well as desktops (unlike Owncloud).

## Commands

Most commands take arguments of the form `--name value`.

### ploppy create

Creates a new network of file-sharing hosts, and starts a server.

* --name

A human-readable name for this network.

* --dir

The directory to share. If it already exists *everything in it will be
shared*. If it doesn't exist it will be created.

* --admin

NYI, email address of the person responsible for the network. Used to notify
him of problems.

* --secret

NYI, a password/key to use for controlling access to the network and
encrypting network traffic.

### plopbox join

Join an existing network and starts a server

* --name

The name of the network to join

* --peer

The ip.ad.re.ss:port pair identifying a peer already on the network,
and which this client can talk to.

* --dir

The directory to use. Must either not exist or be empty.

* --secret

As 'ploppy create'

### ploppy leave

Leaves the network and stops the server

* --name

The name of the network to leave, if the client is on more than one

* --delete

Takes no value, delete the shared directory on leaving the network.

### ploppy network

Show all hosts on the network.

### ploppy status

List all files that this host knows about and their transfer status

### ploppy resolve

Resolve conflicting changes, NYI

## Protocol

All data is transferred using ZeroMQ. There is no particular allocated port.
Clients will pick their own port and notify peers which one they are using.
The protocol is aggressively asynchronous. For example, when a user creates a
new file on a host, the host will announce that to its peers, but won't send it
until asked. Messages are transmitted as a JSON hash and typically have no
immediate response.

Messages always have a 'command' field and an 'id' field. The former is one of the commands listed below. The latter is a UUID, which is passed on to peers
in the event that a message needs to cascade around the network. It is
used to detect and prevent infinite loops. command ids are cached for a
few minutes.

[should we add timestamps? this requires high precision and that all
hosts have their clocks set correctly]

### Network discovery

* HELLO

```json
{
  "name": "The name of the network the host is on",
  "uuid": "to disambiguate when there are multiple NATted networks involved",
  "address: "192.168.0.1:61582",
}
```

It should result in the recipient generating a cascade of 'ICANSEE's and 'IKNOWABOUT's.

* ICANSEE

This is exactly the same as HELLO except that it is announcing the
existence of another peer that the host can talk to directly instead of
announcing its own existence. It should result in a cascade of 'IKNOWABOUT's.

* IKNOWABOUT

As ICANSEE, except that it is a host that the peer can *not* talk to
directly but has merely heard about from other peers. The 'id' field is
used to prevent infinite recursion. It should result in a cascade of 'IKNOWABOUT's.

* ICANTSEE

As ICANSEE, except that it is an announcement that a peer has gone away. It should result in a cascade of 'ICANTSEE's.

* PING

No arguments, just used to periodically verify that a network connection is up. If it fails then an ICANTSEE should be sent to each immediate peer.

### File change notification

* LIST

No extra data. Request that the peer dump its brane, resulting in a slew of IHAVEs.

* IHAVE

```json
  {
    "type": "file", # or directory
    "mode": [number from 0 to 7],
    # the following for files only
    "blocksize": [typically 1024]
    "blocks: [
      "hash1",
      "hash2",
      ...
    ]
  }
```

Note that the mode is only from 0 to 7 - we're only interested in the
owner's permissions, because groups and other users don't translate well
between machines, and this is intended to be run as a non-root user.

* IMOVED

```json
  {
    "oldpath": "foo/bar/baz",
    "newpath": "barf/quux"
  }
```

* IDELETED

```json
  { "oldpath": "foo/bar/baz" }
```

### Block retrieval

* IWANT

```json
  { "hashes": [ "hash1", "hash2", ... ] }
```

This will result in a series of HEREISes

* HEREIS

```json
  { "block": "lots of data" }
```

## Daemon architecture

The daemon immediately forks into two. They communicate by creating
files in the hidden `.ploppy` directory inside the repository. One
process listens for incoming messages from other peers (we'll call this
the server) and the other sends messages (the client).

### Server

For most commands all it will do is leave instructions for the client.
The one exception is when it receives a `HEREIS` command, in which case
it stores the incoming block in a temp file, and, if it now has a complete
file, writes the file to the repository.

It will also listen asynchronously [NB this may become a third fork] for
changes to the repository, using something like `File::ChangeNotify`
[NB that may require writing an FSEvents thingy for F::CN]. Obviously
ignoring the `.ploppy` subdir. When it spots a change it will leave
instructions for the client.

### Client

This listens for changes to the `.ploppy` directory (which will all be
file creates; deletes will be caused by the client removing completed
tasks) and sends appropriate instructions to all peers.
