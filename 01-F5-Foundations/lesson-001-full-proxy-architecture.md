# Lesson 001: BIG-IP Full-Proxy Architecture

## Objective

Understand why F5 BIG-IP is described as a full proxy and how its traffic-processing model differs from conventional packet forwarding.

## Core Principle

A BIG-IP system does not simply forward a client's existing connection to a server.

It:

1. Terminates the client-side connection.
2. Evaluates the traffic using the assigned configuration and policies.
3. Establishes a separate server-side connection.
4. Proxies application data between the two connections.

## Connection Model

```text
Client-side connection:

Client <------> BIG-IP

Server-side connection:

BIG-IP <------> Pool Member
```

These are separate TCP sessions that may have different:

- Source and destination addresses
- TCP sequence numbers
- TCP profiles
- Timeouts
- SSL settings
- Protocol behavior
- Connection lifetimes

## Why This Matters

The full-proxy architecture allows BIG-IP to:

- Terminate and re-encrypt TLS
- Apply different client-side and server-side profiles
- Perform load balancing
- Insert or modify HTTP headers
- Maintain persistence
- Translate addresses with SNAT
- Inspect application traffic
- Protect servers from direct client connections

## Initial Questions

1. Where does the client's TCP connection terminate?
2. Who creates the TCP connection to the selected pool member?
3. Can client-side and server-side SSL settings be different?
4. Does the server normally communicate directly with the original client?
5. Why might packet captures look different on each side of BIG-IP?

## Consultant Perspective

When troubleshooting an application through BIG-IP, always identify which side of the proxy is failing:

- Client to BIG-IP
- BIG-IP traffic processing
- BIG-IP to server

Avoid describing the entire flow as one connection.