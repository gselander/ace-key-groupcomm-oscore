---
title: ACE profile for joining OSCOAP multicast groups
abbrev: Group OSCOAP joining for ACE
docname: draft-tiloca-ace-oscoap-joining-00
# date: 2017-04-25
category: std

ipr: trust200902
area: Security
workgroup: ACE Working Group
keyword: Internet-Draft

# stand_alone: yes

coding: us-ascii

pi:    # can use array (if all yes) or hash here
#  - toc
#  - sortrefs
#  - symrefs
toc: yes
sortrefs:   # defaults to yes
symrefs: yes


author:
 -
    ins: M. Tiloca
    name: Marco Tiloca
    org: RISE SICS AB
    street: Isafjordsgatan 22
    city: Kista
    code: SE-164 29 Stockholm
    country: Swedenn
    phone: +46 70 604 65 01
    email: marco.tiloca@ri.se

 -
    ins: J. Park
    name: Jiye Park
    org: Universitaet Duisburg-Essen
    street: Schuetzenbahn 70
    city: Essen
    code: 45127
    country: Germany
    phone: +49 201 183-7634
    email: ji-ye.park@uni-due.de

normative:
  RFC2119:
  RFC7252:
  I-D.ietf-core-object-security:
  I-D.tiloca-core-multicast-oscoap:
  I-D.ietf-ace-actors:
  I-D.ietf-ace-oauth-authz:
  I-D.seitz-ace-oscoap-profile:
  I-D.gerdes-ace-dtls-authorize:

informative:
  RFC4279:
  RFC6347:
  RFC6749:
  RFC7228:
  RFC7231:
  RFC7250:
  RFC7390:
  I-D.ietf-cose-msg:
  I-D.selander-ace-cose-ecdhe:

--- abstract

This document describes a profile of the ACE framework for Authentication and Authorization. The profile delegates the authentication and authorization of a client willing to join a multicast group where communication is based on CoAP and secured with Object Security of CoAP (OSCOAP). The profile establishes a secure communication channel between the client and the resource server, which is responsible for the multicast group and the joining of new members. The client and resource server leverage protocol-specific profiles for ACE to achieve communication security, proof of possession and server authentication.

--- middle

# Introduction {#sec-introduction}

The Constrained Application Protocol (CoAP) {{RFC7252}} supports also group communication scenarios, where request messages can be delivered to multiple recipients using CoAP on top of IP multicast {{RFC7390}}.

Object Security of CoAP (OSCOAP) {{I-D.ietf-core-object-security}} is a method for application layer protection of CoAP messages, using the CBOR Object Signing and Encryption (COSE) {{I-D.ietf-cose-msg}}, and enabling end-to-end security of CoAP payload and options.

OSCOAP may also be used to protect group communication for CoAP over IP multicast, as described in {{I-D.tiloca-core-multicast-oscoap}}. This relies on a Group Manager entity, which is responsible for managing a multicast group where members exchange CoAP messages secured with OSCOAP. In particular, the Group Manager coordinates the join process of new group members and can be responsible for multiple groups.

This document specifies a profile of the ACE framework for Authentication and Authorization {{I-D.ietf-ace-oauth-authz}}. In this profile, a client willing to join an OSCOAP multicast group securely communicates with a resource server acting as the Group Manager responsible for that group. The access to the resource server concerns a specific resource associated with the multicast group to join and is the first step of the join process.

The client acting as joining node uses an access token bound to a proof-of-possession key to authorize its access to the resource server. The client and resource server leverage protocol-specific profiles for ACE such as {{I-D.seitz-ace-oscoap-profile}} and {{I-D.gerdes-ace-dtls-authorize}} in order to achieve communication security, proof of possession and server authentication.

## Terminology {#ssec-terminology}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in {{RFC2119}}. These words may also appear in this document in lowercase, absent their normative meanings.

Readers are expected to be familiar with the terms and concepts described in the ACE framework for authentication and authorization {{I-D.ietf-ace-oauth-authz}}. Message exchanges are presented as RESTful protocol interactions, for which HTTP {{RFC7231}} provides useful terminology.

The terminology for entities in the considered architecture is defined in OAuth 2.0 {{RFC6749}} and {{I-D.ietf-ace-actors}}. In particular, this includes client (C), resource server (RS), and authorization server (AS). Terminology for constrained environments, such as "constrained device", "constrained-node network", is defined in {{RFC7228}}.

Readers are expected to be familiar with the terms and concepts related to the CoAP protocol described in {{RFC7252}}{{RFC7390}}. Note that the term "endpoint" is used here following its OAuth definition, aimed at denoting resources such as /token and /introspect at the AS and /authz-info at the RS. This document does not use the CoAP definition of "endpoint", which is "An entity participating in the CoAP protocol".

Readers are expected to be familiar with the terms and concepts for protection and processing of CoAP messages through OSCOAP {{I-D.ietf-core-object-security}} also in group communication contexts {{I-D.tiloca-core-multicast-oscoap}}; and with the OSCOAP profile for ACE described in {{I-D.seitz-ace-oscoap-profile}}.

Readers are expected to be familiar with the terms and concepts related to the DTLS protocol {{RFC6347}}; the support for DTLS handshake based on Raw Public Keys (RPK) {{RFC7250}} and on Pre-Shared Keys (PSK) {{RFC4279}}; and the DTLS profile for ACE {{I-D.gerdes-ace-dtls-authorize}}.

This document refers also to the following terminology.

* Joining node: a network node willing to join an OSCOAP multicast group, where communication is based on CoAP and secured with OSCOAP as described in {{I-D.tiloca-core-multicast-oscoap}}.

* Join process: the process through which a joining node becomes a member of an OSCOAP multicast group. The join process is enforced and assisted by the Group Manager responsible for that group.

* Joining endpoint: an endpoint at a Group Manager denoting a resource that an authorized joining node accesses, in order to join the related group under the control of that Group Manager.

# Protocol Overview {#sec-protocol-overview}

TBD

# Joining Node to Authorization Server {#sec-joining-node-to-AS}

TBD

# Joining Node to Group Manager {#sec-joining-node-to-GM}

TBD

# Alternative Key Establishment Modes {#sec-alternative-key-establishment-modes}

TBD

# Updating Authorization Information {#sec-updating-authorization-information}

TBD

# Security Considerations {#sec-security-considerations}

TBD

# IANA Considerations {#sec-iana}

This document has no actions for IANA.

# Acknowledgments {#sec-acknowledgments}

TBD

--- back
