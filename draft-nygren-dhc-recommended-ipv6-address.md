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

This document defines a new DHCPv6 option for communicating one or more recommended /128 IPv6 address to hosts within an assigned prefix. The Recommended Address option allows DHCPv6 servers to suggest specific IPv6 addresses that hosts should additionally use when configuring addresses within the assigned prefix.

--- middle

# Introduction

IA_PD within DHCPv6 {{RFC8415}} allows clients such as hosts to request IPv6 prefixes, typically for delegation to downstream networks or interfaces. In scenarios such as Unique Prefix Per Host {{RFC8273}} the host is given the entire prefix and is free to use addresses from it as it sees fit.

This document defines the Recommended Address option, which can be included within OPTION_IAPREFIX options to suggest one or more specific IPv6 addresses that clients should configure when using the associated prefix. These do not preclude the client from using other addresses within the prefix, such as for temporary addressing.

This is intended for use in managed environments such as datacenters and cloud providers where the operator is configuring a host that they wish to manage or direct clients to. By providing a recommended address, the operator can encourage the host to have a particular /128 address that can be used for management purposes or as a service endpoint. At the same time, the remainder of the prefix is available for the host to use as it sees fit, such as for containers. For example, this allows the customer of a cloud provider to get a full /64 for use by a host while also allowing the customer to configure a specific /128 within that /64 that they can use for managing the host.

The Recommended Address(es) continue to be here are for use by the host, differentiating this from PD\_EXCLUDE specified in {{RFC6603}}.

The Recommended Address option is only advisory and while clients MAY use these /128 addresses they are not required to do so.

# Requirements Language

{::boilerplate bcp14-tagged}

# Recommended Address Option

The Recommended Address option provides a mechanism for DHCPv6 servers to suggest one or more specific IPv6 addresses within an assigned prefix that host clients should use on the their interface.

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
~~~

Where:

option-code:
: OPTION_RECADDR (TBD by IANA)

option-len:
: 17 (length of the option data in octets)

recommended-address:
: A 128-bit IPv6 address that the server recommends the client use

## Option Usage

The Recommended Address option MUST only appear as a sub-option within an OPTION\_IAPREFIX option. Multiple Recommended Address options MAY be included within a single OPTION\_IAPREFIX option to suggest that multiple addresses within the prefix should be assigned by the host to its interface.

The recommended-address field MUST contain an IPv6 address that falls within the prefix specified by the enclosing OPTION_IAPREFIX option. Servers MUST NOT include Recommended Address options with addresses outside the associated prefix.

# DHCPv6 Server Behavior

DHCPv6 servers MAY include OPTION\_RECADDR within OPTION\_IAPREFIX options in ADVERTISE, REPLY, and RENEW responses.

Servers SHOULD validate that any recommended addresses fall within the prefix bounds of the enclosing OPTION\_IAPREFIX option before including them in responses.

Servers SHOULD avoid providing recommended addresses that would fall within those assigned to EUI-64 SLAAC addresses.

Servers MUST NOT require clients to use recommended addresses, although operators of some tightly managed environments may set expectations that accepting connections on recommended address is required for proper operational function. Operators MUST NOT assume that clients will use only recommended addresses as source addresses from within the prefix as clients remain free to use the entire delegated prefix for outbound connections.

# DHCPv6 Client Behavior

DHCPv6 clients MAY process Recommended Address options received within OPTION_IAPREFIX options. Clients that do not understand or support the Recommended Address option MUST ignore it, as per standard DHCPv6 option processing rules.

Clients configured to accept Recommended Addresses SHOULD configure the Recommended Addresses on a local host interface such that local services are reachable on that address.

Prior to using the address, clients MUST validate that recommended address(es) fall within the bounds of the associated prefix and any outside MUST be ignored.

Clients MAY also assign SLAAC addresses such as temporary addresses within the prefix. While clients MAY use Recommended Addresses as a preferred source address they are not required to do so.

## Address Lifetime and Removal

The lifetime of the Recommended Addresses are associated with that of the containing OPTION\_IAPREFIX and its associated lease. Clients SHOULD set the valid lifetime and preferred lifetime ({{RFC4862}}) for Recommended Addresses to the remaining lifetime of the DHCPv6 lease associated with the OPTION\_IAPREFIX.

If a client had previously received a Recommended Address for a prefix but an subsequent advertisement for the same OPTION\_IAPREFIX no longer contains it, the client SHOULD deprecate the address from its interface, such as by setting the preferred-lifetime of the address to 0 but leave the valid-lifetime as the remaining lifetime of the associated DHCPv6 lease.


# Relationship to Prefix Exclude Option

Unlike {{RFC6603}} which specifies a sub-prefix to exclude from the delegated prefix, Recommended Addresses propose one or more recommended addresses that the host client should use. Servers MUST NOT suggest a recommended address within a OPTION\_PD\_EXCLUDE prefix.

To be removed prior to publication:
An alternate proposal was to use IA\_NA within an excluded prefix, but that was thought to be too messy.

# Use-Cases and Suitability

Cases where an operator may choose to deploy as an alternative to using IA\_NA:

* Client hosts only support IA\_PD and not IA\_NA but the operator wishes to continue to have at least one known /128 address on the host. 
* Client hosts that support IA\_PD (such as IPv6 CE routers) and which also need an address on which they can be managed.

The alternatives for having both a IA\_PD /64 and an IA\_NA /128 for a client host is to use either a larger /63 prefix (with half of it only being used sparselyu for the /128) or to allocate the /64 and /128 from disjoint space. This latter scenario increases FIB count.  Both of these alternatives require clients to support both IA\_PD and IA\_NA.

# Security Considerations

The Recommended Address option is subject to the same authentication and security mechanisms as the base DHCPv6 protocol. Deployments requiring authentication SHOULD use DHCPv6 authentication mechanisms {{RFC3315}} or secure the DHCPv6 communication channel.

Recommended Addresses may have less entropy (or otherwise be more predictable) than those assigned via other mechanisms such as {{RFC4941}} and as such may be easier for attackers to find while scanning IPv6 address space.

# Privacy Considerations

Recommended Addresses are primarily intended for use in managed environments such as data centers and cloud providers.

This option extends upon DHCPv6 so has similar privacy properties to other modes of DHCPv6 such as IA_NA. See {{RFC7824}} for an extensive discussion of these properties. As Recommended Addresses are under server control and may have less entropy, they may be more predictable within the /64 than addresses under the clients control to select. Clients configured to use {{RFC4941}} Privacy Addresses or similar scheme may chose to prefer those as a default source address.


# IANA Considerations

This document requests IANA to assign a new DHCPv6 option code for the Recommended Address option from the "Dynamic Host Configuration Protocol for IPv6 (DHCPv6)" registry.

| Option Name | Value | Description | Reference |
|-------------|-------|-------------|-----------|
| Recommended Address | TBD | Recommended IPv6 address within an assigned prefix | This document |

--- back

# Acknowledgements

Thank you to Lorenzo Colitti, Kasper Dupont, and others for their feedback and suggestions on this document.

# Open Questions

NOTE TO RFC EDITOR: to be removed before publication.

Some open questions for discussions include:

* Should we allow prefixes in-addition to /128 along with labels for those prefixes? For example to signal things like which ranges should be sub-delegated for use by Docker, etc?
* What name should we use for this option?  OPTION\_RECADDR? OPTION\_RECOMMENDED\_ADDRESS?
* When should we use "client" vs "host"?
* Do we need more description of use-cases and purpose?

# Change History

NOTE TO RFC EDITOR: to be removed before publication.

## Changes in -01

* Clarify handling of removal, as well as provide recommendations for address lifetime.
* Add use-cases and suitability. Increase clarity on the use-cases for hosts as well as the alternative.

## Prior to -00

* Removed priority and made it very clear that this is in-addition to temporary addresses and does not require clients to use these as a source address.

