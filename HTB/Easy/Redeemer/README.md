# Redeemer — HackTheBox Write-up

**Difficulty:** Easy
**OS:** Linux
**Category:** Database Misconfiguration (Redis)

---

## Overview

Redeemer is an Easy-rated box built around a single, very common misconfiguration: a Redis instance exposed to the network with no authentication at all. Once the open port is identified, the entire path to the flag is just a matter of connecting with `redis-cli` and poking around with a handful of built-in Redis commands. It's a good introduction to how in-memory databases can become an easy foothold when they're left open by default.

## Recon

### Nmap

```
nmap -p- -sC -sV <TARGET_IP>
```

A full port scan revealed a single open TCP port:

```
6379/tcp open  redis
```

Port **6379** is the default port for **Redis**, an in-memory data store commonly used for caching and message brokering.

### Enumeration

#### Redis — Port 6379

Redis ships with its own command-line client, `redis-cli`, which is the standard way to interact with a Redis server. Connecting to the target's Redis instance was as simple as:

```
redis-cli -h <TARGET_IP>
```

The `-h` flag specifies the host to connect to. No authentication was required — the server accepted the connection immediately, which was the first sign of a misconfigured, publicly exposed database.

Once connected, the `INFO` command was used to gather details about the server:

```
INFO
```

This returned server statistics including the Redis version, memory usage, connected clients, and configuration details. The version identified was:

```
5.0.7
```

## Foothold

With a live, unauthenticated connection to the Redis server, the next step was to look for data stored inside it. Redis organizes data into numbered databases, selected with the `SELECT` command:

```
SELECT 0
```

Once inside database 0, all stored keys were listed with:

```
KEYS *
```

This returned 4 keys total. Among them was a key named `flag`, which was retrieved directly with the `GET` command:

```
GET flag
```

## Root / System

The `GET flag` command returned the flag value directly from the database — no further privilege escalation was needed. The exposed, unauthenticated Redis instance gave immediate access to sensitive data stored in memory.

## Vulnerabilities Summary

| Category | Vulnerability | Why It Mattered |
|---|---|---|
| Authentication | Redis server exposed with no password/authentication configured | Allowed anyone with network access to connect and run arbitrary Redis commands |
| Configuration | Default port (6379) reachable from outside the intended network boundary | Made the service trivially discoverable via a standard port scan |
| Information Disclosure | Sensitive data (the flag) stored as a plaintext key in the database | Exposed data directly retrievable with a single `GET` command once connected |

## Lessons Learned / Takeaways

- **Not every foothold needs an exploit.** Redis has no authentication enabled by default — if it's exposed without a `requirepass` configuration, anyone who can reach the port has full access to the data and, in many real-world cases, to much more (e.g., writing SSH keys via Redis' persistence features).
- **`redis-cli` basics are worth knowing cold**: `-h` for host, `INFO` for server stats, `SELECT <db>` to switch databases, and `KEYS *` to enumerate stored data. These four commands alone cover most of the initial recon needed on a misconfigured Redis instance.
- **A full port scan (`-p-`) matters.** With only one service running on a non-standard scope of ports, a default top-1000 scan could have missed context — running a full scan is a good habit on boxes that seem unusually quiet.
- This box is a solid, simple reminder that database services are frequently deployed with insecure defaults, and that "no authentication configured" is still one of the most common real-world findings in internal network assessments.

## References

- [Redis Security Documentation](https://redis.io/docs/latest/operate/oss_and_stack/management/security/)
- [redis-cli documentation](https://redis.io/docs/latest/develop/tools/cli/)
