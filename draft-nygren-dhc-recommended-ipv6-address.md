---
title: "DHCPv6 Recommended IPv6 Address Option"
abbrev: "DHCPv6 Recommended Address"
docname: draft-nygren-dhc-recommended-ipv6-address-latest
category: std

ipr: trust200902
keyword: Internet-Draft

stand_alone: yes
stream: IETF
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 -
    fullname: Erik Nygren
    organization: Akamai Technologies
    email: erik+ietf@nygren.org

normative:
  RFC2119:
  RFC8174:
  RFC8415:

informative:
  RFC3315:
  RFC6603:
  RFC7824:
  RFC8273:
  RFC4941:

--- abstract

This document defines a new DHCPv6 option for communicating one or more recommended /128 IPv6 address to hosts within an assigned prefix. The Recommended Address option allows DHCPv6 servers to suggest specific IPv6 addresses that hosts should preferentially use when configuring addresses within the assigned prefix.

--- middle

# Introduction

IA_PD within DHCPv6 {{RFC8415}} allows clients such as hosts to request IPv6 prefixes, typically for delegation to downstream networks or interfaces. In scenarios such as Unique Prefix Per Host {{RFC8273}} the host is given the entire prefix and is free to use addresses from it as it sees fit.

This document defines the Recommended Address option, which can be included within OPTION_IAPREFIX options to suggest one or more specific IPv6 addresses that clients should preferentially configure when using the associated prefix.

This is intended for use in managed environments such as datacenters and cloud providers where the operator is configuring a host that they wish to manage or direct clients to. By providing a recommended address, the operator can encourage the host to have a particular /128 address that can be used for management purposes or as a service endpoint. At the same time, the remainder of the prefix is available for the host to use as it sees fit, such as for containers.

The Recommended Address option is only advisory and while clients MAY use these /128 addresses they are not required to do so.

# Requirements Language

{::boilerplate bcp14-tagged}

# Recommended Address Option

The Recommended Address option provides a mechanism for DHCPv6 servers to suggest one or more specific IPv6 addresses within an assigned prefix that host clients should use on the their interface.

When multiple addresses are provided, a priority value indicates which should be preferred as a default source address, enabling operational transitions such as renumbering.

## Option Format

The format of the Recommended Address option is:

~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|      OPTION_RECADDR           |           option-len          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|                     recommended-address                       |
|                                                               |
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  priority   |
+-+-+-+-+-+-+-+
~~~

Where:

option-code:
: OPTION_RECADDR (TBD by IANA)

option-len:
: 17 (length of the option data in octets)

recommended-address:
: A 128-bit IPv6 address that the server recommends the client use

priority:
: An 8-bit unsigned integer indicating the relative priority for this recommended address to be used as the default source address. Higher values indicate higher priority.

## Option Usage

The Recommended Address option MUST only appear as a sub-option within an OPTION\_IAPREFIX option. Multiple Recommended Address options MAY be included within a single OPTION\_IAPREFIX option to suggest that multiple addresses within the prefix should be assigned by the host to its interface.

The recommended-address field MUST contain an IPv6 address that falls within the prefix specified by the enclosing OPTION_IAPREFIX option. Servers MUST NOT include Recommended Address options with addresses outside the associated prefix.

The priority field allows servers to indicate relative preferences when multiple addresses are recommended. Priority values are relative only within the context of a single OPTION_IAPREFIX option.

# DHCPv6 Server Behavior

DHCPv6 servers MAY include OPTION\_RECADDR within OPTION\_IAPREFIX options in ADVERTISE, REPLY, and RENEW responses.

Servers SHOULD validate that any recommended addresses fall within the prefix bounds of the enclosing OPTION\_IAPREFIX option before including them in responses.

When including multiple OPTION\_RECADDR options within a single OPTION_IAPREFIX option, servers SHOULD assign the highest priority value to the option that they wish to see in active use as a default source address. Changing the primary recommended address may be performed by introducing a new address with a lower priority value, then by switching it to be the highest priority, and then by removing the old address.

Servers SHOULD avoid providing multiple addresses with the same priority.

Servers SHOULD avoid providing recommended addresses that would fall within those assigned to EUI-64 SLAAC addresses.

Servers MUST NOT require clients to use recommended addresses, although operators of some tightly managed environments may set expectations that using the highest priority recommended address is required for proper operational function.

# DHCPv6 Client Behavior

DHCPv6 clients MAY process Recommended Address options received within OPTION_IAPREFIX options. Clients that do not understand or support the Recommended Address option MUST ignore it, as per standard DHCPv6 option processing rules.

Clients configured to accept Recommended Addresses SHOULD configure the Recommended Addresses on a local host interface such that local services are reachable on that address.

Prior to using the address, clients MUST validate that recommended address(es) fall within the bounds of the associated prefix and any outside MUST be ignored.

If a client had previously received a Recommended Address for a prefix but an subsequent advertisement for the same OPTION\_IAPREFIX no longer contains it, the client SHOULD remove the address from its interface.

When multiple Recommended Address options are present within an OPTION_IAPREFIX option, clients SHOULD use the one with the highest priority as the default source address. If more than two recommended addresses are provided, clients MAY choose to configure only the two with the highest priority.

Clients MAY also assign SLAAC addresses within the prefix, but the highest priority recommended address should be given preference for default source address selection.

# Relationship to Prefix Exclude Option

Unlike {{RFC6603}} which specifies a sub-prefix to exclude from the delegated prefix, Recommended Addresses propose one or more recommended addresses that the host client should use. Servers MUST NOT suggest a recommended address within a OPTION\_PD\_EXCLUDE prefix.

To be removed prior to publication:
An alternate proposal was to use IA\_NA within an excluded prefix, but that was thought to be too messy.

# Security Considerations

The Recommended Address option is subject to the same authentication and security mechanisms as the base DHCPv6 protocol. Deployments requiring authentication SHOULD use DHCPv6 authentication mechanisms {{RFC3315}} or secure the DHCPv6 communication channel.

Recommended Addresses may have less entropy (or otherwise be more predictable) than those assigned via other mechanisms such as {{RFC4941}} and as such may be easier for attackers to find while scanning IPv6 address space.

# Privacy Considerations

Recommended Addresses are primarily intended for use in managed environments such as data centers and cloud providers.

This option extends upon DHCPv6 so has similar privacy properties to other modes of DHCPv6 such as IA_NA. See {{RFC7824}} for an extensive discussion of these properties. As Recommended Addresses are under server control and may have less entropy, they may be more predictable within the /64 than addresses under the clients control to select.


# IANA Considerations

This document requests IANA to assign a new DHCPv6 option code for the Recommended Address option from the "Dynamic Host Configuration Protocol for IPv6 (DHCPv6)" registry.

| Option Name | Value | Description | Reference |
|-------------|-------|-------------|-----------|
| Recommended Address | TBD | Recommended IPv6 address within an assigned prefix | This document |

--- back

# Open Questions

NOTE TO RFC EDITOR: to be removed before publication.

Some open questions for discussions include:

* Is the priority useful?  Should we call it priority or preference?  Should it be 8-bits or just a boolean flag? Should we remove it altogether?
* Would we want flags instead of priority?
* Should we allow prefixes in-addition to /128 along with labels for those prefixes? For example to signal things like which ranges should be sub-delegated for use by Docker, etc?
* What name should we use for this option?  OPTION\_RECADDR? OPTION\_RECOMMENDED\_ADDRESS?
* When should we use "client" vs "host"?
* Do we need more description of use-cases and purpose?
