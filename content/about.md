---
layout: default
title: About
permalink: /about/
nav_order: 2
---

## AS, RIR, and BGP
---------

_The following is introductory information on Autonomous Systems (AS) and BGP, as well as prior art on ASmap. A [Glossary](/glossary) is provided._

The allocation of IP address space on the Internet begins with the *Internet Assigned Numbers Authority* (IANA), which allocates IP address ranges to a *Regional Internet Registry* (RIR). This RIR then assigns IP address ranges to an AS they control. They then announce to the world which AS controls which IP address range via the *Border Gateway Protocol* (BGP).

Governments around the world have their own reserved ranges, as do *Internet Service Providers* (ISP) and some commercial entities. For example, Amazon has their own AS, as does Cloudflare, and therefore each has control over IP addresses in a given range. Cloudflare has a [good explainer](https://www.cloudflare.com/learning/network-layer/what-is-an-autonomous-system/) about AS and where they fit in the Internet’s architecture. Each AS is given a number to be identified by.

### The Problem

The central problem is that sharing a route via BGP relies on trust, in the sense that an AS implicitly trusts the routes that are shared with them. We are presented with the challenge that _the root address space protocol of the Internet has no measures against malicious announcements or errors_, and so an important part of the ASmap project is to develop heuristics to parse the BGP data correctly, to the extent possible.

 Bitcoin Core began implementing some measures to address eclipse attacks several years ago. From Gleb Naumenko's July 2020 "Call to Action" [post](https://blog.bitmex.com/call-to-action-testing-and-improving-asmap/):

> "To increase the cost of these attacks, Bitcoin Core started diversifying peer connections by what is called a netgroup bucketing. For instance, if you take a simple IP address like “172.16.254.1”, Bitcoin Core connects to at most one IP which looks like “172.16.any.any” (if possible).

> The assumption was that it would be more difficult for an attacker to create fake nodes in different netgroups. This was based on the expectation that netgroups roughly correspond to regions and internet providers, therefore running fake nodes would require negotiating with many actors and make bulk deals less useful.

> [...] Unfortunately, the assumption that netgroups correspond to regions and internet providers no longer holds. Over the past years, IPv4 addresses have become more fluid, in the sense that they are traded between entities and resulting mapping is now in many cases near-random. For example, Amazon now controls many IP ranges."

Starting with Core 20.0, you can pass an ASmap file via the `-asmap=<filepath>` option. Sourcing the data and generating these ASmaps require some tooling.

Beginning with Core v31, an ASmap is embedded directly with the release binary, but the feature remains off by default. To enable it (as of September 2026), pass `-asmap=1` or pass an explicit filepath `-asmap=/path/to/asmap/dat`.

## ASmap in Bitcoin Core

---------------------

Using an ASmap file in one's Bitcoin Core configuration is already strictly better than the default, and so we want to encourage usage of ASmap among users.

## Usage

`bitcoind` will accept a compressed ASmap file with the `-asmap` startup option.
You can download a pre-made [latest_asmap.dat](https://github.com/fjahr/asmap-data/blob/main/latest_asmap.dat) file in the [asmap-data](https://github.com/fjahr/asmap-data) repo. The `latest_asmap.dat` generated prior to a release cutoff date is embedded into the Core release.

### Create an ASmap with Kartograf

You can choose to generate an ASmap file yourself. [Kartograf](https://github.com/fjahr/kartograf) is a tool that fetches AS data from multiple sources, combines them, and produces a file with raw map data that can be used in Bitcoin Core after being compressed.

### Compress it with `asmap-tool`

If you generate a file yourself, you must compress it before passing it to `bitcoind`. [asmap-tool](https://github.com/bitcoin/bitcoin/tree/master/contrib/asmap) is a Python script to help encode/compress an ASmap file. `asmap-tool` is included in the Bitcoin Core repository.

### ASmap Health Check

If an ASmap is provided when starting bitcoind, a health check will run during startup and then every 24 hours. It logs the level of coverage the ASmap provides for all the clearnet addresses known to our node. For example:
```
ASMap Health Check: 32546 clearnet peers are mapped to 3127 ASNs with 113 peers being unmapped
```
Meaning, there are AS mappings available for the IPs of 32546 of our peers. 113 peers don't have an AS mapping for their IP in the provided ASmap.

### ASmap fields in RPC

If ASmap is enabled, the `getpeerinfo` RPC command's response will include `mapped_as` field, indicating which AS this peer's IP was mapped to via the given ASmap, if any.

Similarly, the `getrawaddrman` RPC command's response will include `source_mapped_as` field, indicating which AS this peer's source IP was mapped to, if any (as of Bitcoin Core [v28.0](https://github.com/bitcoin/bitcoin/blob/1147e72953d1f262111a4b1d5a438a8394511bc7/src/rpc/net.cpp#L1160)).

## Prior work

fjahr, brunoerg, naumekogs, and sipa contributed much of the work behind integrating ASmap data in Core.

------------------
