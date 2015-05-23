# ploppy

Like Dropbox and Owncloud, but better

## Description

ploppy is a peer-to-peer file replication and distribution system which has no centralised database (unlike Dropbox) and works on servers as well as desktops (unlike Owncloud).

## Commands

Most commands take arguments of the form `--name value`.

## ploppy create

Creates a new network of file-sharing hosts, and starts a server.

### --name

A human-readable name for this network.

### --dir

The directory to share. If it already exists *everything in it will be
shared*. If it doesn't exist it will be created.

### --admin

NYI, email address of the person responsible for the network. Used to notify
him of problems.

### --secret

NYI, a password/key to use for controlling access to the network and
encrypting network traffic.

## plopbox join

Join an existing network and starts a server

### --name

The name of the network to join

### --peer

The ip.ad.re.ss:port pair identifying a peer already on the network,
and which this client can talk to.

### --secret

As 'ploppy create'

## ploppy leave

Leaves the network and stops the server

### --name

The name of the network to leave, if the client is on more than one

### --delete

Takes no value, delete the shared directory on leaving the network.

## ploppy network

Show all hosts on the network.

## ploppy status

List all files that this host knows about and their transfer status

## ploppy resolve

Resolve conflicting changes, NYI
