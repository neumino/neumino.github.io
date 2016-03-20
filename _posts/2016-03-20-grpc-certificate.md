---
layout: post
category : Geek
tags : [grpc, certificate]
title: Certificates for GRPC
---
{% include JB/setup %}

I haven't blogged anything for the past few months because I have been busy
working on a few projects using new shiny toys. One of them is [gRPC](https://www.grpc.io), a high
performance, open source, general RPC framework, which is based on Google's
internal Stubby RPC system.

This post is mostly about using TLS with gRPC in Golang, but if you are wondering
what gRPC brings that HTTP/JSON does not, here are a few reasons on top of my head:

- Better performance in respect to network compression, serialization etc.
- Structured RPC.
- Error tracing.

Anyway, if you want to use TLS with gRPC, you need to create a few certificates first.
This script does it for you. Usage is `gen.sh <host> <prefix>` where `host` is the `host`
on which your gRPC server is listening. The parameter `prefix` is just used to prefix the
output files such that you can create prod and dev certificates.

```bash
PASSWORD=`cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1`
HOST=$1
if [ -z $HOST ]; then HOST="localhost"; fi
PREFIX=$2
if [ -z $PREFIX ]; then PREFIX="dev"; fi

openssl genrsa -passout pass:$PASSWORD -des3 -out $PREFIX'server.key' 4096
openssl req -passin pass:$PASSWORD -new -x509 -days 3650 -key $PREFIX'server.key' -out $PREFIX'server.crt' -subj '/C=US/ST=CA/L=Sunnyvale/O=MyApp/CN='$HOST'/emailAddress=foo@bar.com'
openssl rsa -passin pass:$PASSWORD -in $PREFIX'server.key' -out $PREFIX'server.key'
```

Two files are created, a `.crt` and `.key` one. You can then create create a connection with:

```go
creds, err := credentials.NewClientTLSFromFile(grpcCrtFile, host)
if err != nil {
  log.Fatalf("Failed to create TLS credentials %v", err)
}
opts := grpc.WithTransportCredentials(creds)
connection, err := grpc.Dial(g.address, g.opts)
```


Create your server with:

```go
lis, err := net.Listen("tcp", *grpcHost+":"+strconv.Itoa(*grpcPort))
if err != nil {
  log.Fatalf("failed to listen: %v", err)
}

creds, err := credentials.NewServerTLSFromFile(grpcCrtFile, grpcKeyFile)
if err != nil {
  log.Fatalf("Failed to generate credentials %v", err)
}

s := grpc.NewServer(grpc.Creds(creds))
// Register your services here
s.Serve(lis)
```
