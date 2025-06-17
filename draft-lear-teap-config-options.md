---
title: "New TEAP TLV for Encapsulating DHCPv6 Options"
abbrev: "TEAP DHCPv6 TLV"
category: std

docname: draft-lear-teap-config-options-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3

area: Security
workgroup: EMU Working Group

keyword:
  - TEAP
  - TLV
  - DHCP
  - Authentication
venue:
  group: EMU WG
  type: Working Group
  mail: emu@ietf.org
  arch: https://example.com/WG
  github: elear/draft-lear-teap-config-option
  latest: https://elear.github.io/draft-lear-teap-config-option/LATEST

author:
  -
    ins: E. Lear
    name: Eliot Lear
    org: Cisco Ssytes, Inc.
    email: lear@cisco.com
    street: Richtistrasse 7
    code: CH-8304
    city: Wallisellen
    country: Switzerland
    phone: +41 44 878 9200
    email: lear@cisco.com
  -
    ins: A. Dekok
    name: Alan Dekok
    org: InkBridge Networks
    email: aland@deployingradius.com

--- abstract

This document defines a new Tunneled Extensible Authentication
Protocol (TEAP) Type-Length-Value (TLV) to encapsulate DHCPv6 (Dynamic
Host Configuration Protocol Version 6) options within TEAP
authentication exchanges. This enhancement enables exchange
of network configuration parameters after the authentication
phase.

--- middle

# Introduction

TEAP (Tunneled Extensible Authentication Protocol), defined in
{{!RFC7170}}, supports the use of Type-Length-Value (TLV) structures
to exchange additional data during the authentication process. DHCPv6
{{!RFC8415}} is widely used for configuration of network
parameters. This document introduces a new TLV to encapsulate DHCPv6
options within TEAP messages.

{{?RFC9445}} specifies a way to communicate options to a radius
client.  This memo specifies a means to communicate options
end-to-end between the authentication server and the peer.

Not all DHCP communications will necessarily make sense in this
context.  For instance, a AAA server may only wish to send
non-topological options, such as a print server, or next hop
configuration URL, and it might not send next-hop router or
IP address binding information.

## Use of both DHCPv6 and TEAP protocols

Because DHCPv6 is widely deployed, peers implementing this specifciation
can expect to receive information via TEAP and DHCPv6.  Thereefore, the
possibility of a conflict arises.  Clients are not in a good position to
determine on their own which information is correct.  Therefore, the
following strategy is RECOMMENDED:

1. Peers receiving information only via DHCP will use that information.
2. Peers receiving DHCP information only via TEAP will use that information.
3. Peers receiving overlapping DHCPv6 options SHOULD select TEAP information
   since it is likely to be better authenticated and unchanged.
4. If conflicting information is received by the peer it SHOULD log the
   conflict as an error and MAY produce an exception.

# TEAP TLV for DHCPv6 Options

Both the TEAP server and TEAP client MAY transmit this TLV during
Phase 2, and it MAY be included in a Request Action frame.  The
Mandatory bit MUST not be set.  If either side sends this TLV prior
to Phase 2, an error TLV of 2002 (Unexpected TLVs Exchanged)
be returned with a Result of Status=Failure.  That is, the table
in Section 4.3.1 of {{!I-D.ietf-emu-rfc7170bis}} remains unchanged.

Thus this TLV MAY be used as follows:

| Request | Response | Success | Failure | TLV          |
|---------|----------|---------|---------|--------------|
|   0+    |   0+     |  0+     |   0     |DHCPv6 options|
{: #tabAllowed title="When is this TLV Allowed"}

A TEAP peer or server receiving this TLV SHOULD NOT act on it until
the other side has been sufficiently authenticated, but it is not an
error to send this TLV in advance of such authentication.  In this
way, the TLV can be conveniently piggybacked as part of the
authentication prior to a result or intermediate result being
generated.

##  TLV Format

The new TEAP TLV for DHCPv6 options is structured as follows:

~~~~~

    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |M|R|         TLV Type          |            Length             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           DHCPv6 fields...
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-++-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-

~~~~~
{: #packetform title="DHCP options TLV format"}

~~~~~
   M

      0 - Optional TLV

   R

      Reserved, set to zero (0)

   TLV Type

      TBD - DHCPv6 options

   Length

      >=2 octets

~~~~~

## DHCPv6 Option Encapsulation

The Value field encapsulates DHCPv6 options exactly as they appear in
DHCPv6 messages. Multiple options can be included, concatenated
sequentially, with no padding between them.

The format adheres to the standard DHCPv6 option encoding, ensuring compatibility with existing DHCPv6 implementations.

Encapsulating this option in the TLV would look as follows (in hexadecimal):

~~~~~
Type: 0xXXXX (assigned by IANA)
Length: 0x0014
Value: 0x00170010FE8000000000000000000000000001
~~~~~

Where:

- `Type = 0xXXXX` (placeholder for the assigned type value)
- `Length = 0x0014` (20 bytes for the "Recursive DNS Servers" option, including addresses)
- `Value = 0x00170010FE8000000000000000000000000001` (DHCPv6 Option 23 with an IPv6 address of `FE80::1`).


# Security Considerations

Encapsulating DHCPv6 options within TEAP messages inherits the
security guarantees of the TEAP protocol. Further details on
mitigation strategies are discussed in {{!RFC7170}}.

# IANA Considerations

IANA is requested to assign a new TEAP TLV type value for the DHCPv6
Options TLV from the TEAP TLV Type registry defined in {{!RFC7170}}.


# Acknowledgements

The authors hope to thank the members of the IETF EMU Working Group
for their valuable input and contributions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
