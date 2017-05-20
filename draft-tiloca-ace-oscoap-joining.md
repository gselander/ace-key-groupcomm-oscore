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

* Joining node: a network node willing to join an OSCOAP multicast group, where communication is based on CoAP {{RFC7390}} and secured with OSCOAP as described in {{I-D.tiloca-core-multicast-oscoap}}.

* Join process: the process through which a joining node becomes a member of a multicast group. The join process is enforced and assisted by the Group Manager responsible for that group.

* Join resource: a protected resource hosted by the Group Manager, associated to a multicast group under that Group Manager. A joining node accesses the join resource in order to start the join process and become a member of that group.

* Join endpoint: an endpoint hosted by the Group Manager associated to a join resource.

# Protocol Overview {#sec-protocol-overview}

Group communication for CoAP over IP multicast has been enabled in {{RFC7390}} and can be secured with OSCOAP as described in {{I-D.tiloca-core-multicast-oscoap}}. A network node explicitly joins an OSCOAP multicast group, by interacting with the responsible Group Manager (GM). Once registered in the group, the new node can securely exchange (multicast) messages with other group members.

This profile describes how a network node joins an OSCOAP multicast group leveraging the ACE framework for authentication and authorization {{I-D.ietf-ace-oauth-authz}}. With reference to the ACE framework and the terminology defined in OAuth 2.0 {{RFC6749}}:

* The Group Manager acts as Resource Server (RS), and owns one join resource for each OSCOAP multicast group it manages. Each join resource is exported by a distinct join endpoint.

* The joining node acts as Client (C), and requests to join the OSCOAP multicast group by accessing the related join endpoint at the Group Manager.

* The Authorization Server (AS) enables and enforces the authorized access of Client nodes to join endpoints at the Group Manager. Multiple Group Managers can be associated to the same AS.

If authorized to join the multicast group, the joining node receives from the AS an Access Token bound with a proof-of-possession key. Then, the joining node provides the RS with the Access Token, in order to access the join endpoint. This step involves the opening of a secure communication channel between the joining node and the Group Manager, in case they have not already established one. Finally, the joining node performs the join process with the Group Manager, in order to become a member of the multicast group.

The AS is not necessarily expected to release Access Tokens for any other purpose than accessing join resources on registered Group Managers. In particular, the AS is not necessarily expected to release Access Tokens for accessing protected resources at members of multicast groups. All communications between the involved entities rely on the CoAP protocol and MUST be secured.

This profiles specifies the following steps to take for joining an OSCOAP multicast group, by leveraging the CoAP-DTLS profile of ACE {{I-D.gerdes-ace-dtls-authorize}} or the OSCOAP profile of ACE {{I-D.seitz-ace-oscoap-profile}}.

1. The joining node retrieves an Access Token from AS to access a join resource on the Group Manager ({{sec-joining-node-to-AS}}). The response from AS enables the joining node to start a secure channel with the Group Manager, if not already established. The joining node can also contact the AS for updating a previously released Access Token, in order to access further groups under the same Group Manager ({{sec-updating-authorization-information}}).

2. Authentication and authorization information are transferred between the joining node and the Group Manager, starting a secure channel if not already established ({{sec-joining-node-to-GM}}). That is, a joining node MUST establish a secure communication channel with a Group Manager, before joining a mulitcast group under that Group Manager for the first time.

3. The joining node accesses the join resource hosted by the Group Manager, and performs the join process to become a member of the multicast group.

Communications between the joining node and AS (/token endpoint) and between the Group Manager and AS (/introspection endpoint) can be secured by different means, e.g. with DTLS {{RFC6347}} or with OSCOAP (see Sections 3 and 4 of {{I-D.seitz-ace-oscoap-profile}}).

# Joining Node to Authorization Server {#sec-joining-node-to-AS}

This section considers a joining node that intends to contact the Group Manager for the first time. That is, the joining node has never attempted before to join a multicast group under that Group Manager. Also, the joining node and the Group Manager do not have a secure communication channel already established.

In case the specific AS associated to the Group Manager is unknown to the joining node, the latter can rely on mechanisms like the one described in Section 2.2 of {{I-D.gerdes-ace-dtls-authorize}} to discover the correct AS in charge of the Group Manager.

The joining node contacts the AS, in order to request an Access Token for accessing the join resource(s) hosted by the Group Manager. In particular, the Access Token request sent to the /token endpoint specifies the join endpoint(s) of interest at the Group Manager and the raw public key of the joining node. In particular, the Access Token request includes a "cnf" object conveying either the raw public key of the joining node, or a unique identifier for a raw public key which has been previously made known to the AS.

The AS is responsible for authorizing the joining node, accordingly to group join policies enforced on behalf of the Group Manager. In case of successful authorization, the AS releases an Access Token bound to the joining node's raw public key as proof-of-possession key.

Then, the AS provides the joining node with the Access Token. Furthermore, the Access Token response includes the raw public key of the Group Manager and indicates how to secure communications with the Group Manager, when accessing the join resource(s) for which the Access Token is valid. In particular, the Access Token response MUST specify one of the following alternatives:

* CoAP over DTLS, i.e. coaps://, indicating to consider the CoAP-DTLS profile of ACE in RawPublicKey Mode, as described in Section 3 of {{I-D.gerdes-ace-dtls-authorize}}.

* OSCOAP, indicating to consider the OSCOAP profile of ACE with asymmetric key as proof-of-possession key, as described in Section 2.2 of {{I-D.seitz-ace-oscoap-profile}}.

# Joining Node to Group Manager {#sec-joining-node-to-GM}

First, the joining node establishes a secure channel with the Group Manager, according to the alternative specified in the Access Token response. In particular:

* If the CoAP-DTLS profile of ACE is considered, the joining node MUST upload the Access Token to the /authz-info endpoint before starting the DTLS handshake. The Group Manager processes the Access Token according to {{I-D.ietf-ace-oauth-authz}} and proceeds according to Section 3 of {{I-D.gerdes-ace-dtls-authorize}}. If this yields to a positive response, the joining node and the Group Manager establishes a DTLS session by performing the handshake in RawPublicKey Mode, as described in Section 4.1 of {{I-D.gerdes-ace-dtls-authorize}}.

* If the OSCOAP profile of ACE is considered, the joining node and the Group Manager establishes an OSCOAP channel by using the EDHOC protocol {{I-D.selander-ace-cose-ecdhe}}, as described in Section 2.2 of {{I-D.seitz-ace-oscoap-profile}}. In particular, the joining node MUST include the Access Token in the EDHOC message_1 sent to the /authz-info endpoint. The Group Manager processes the Access Token as specified in {{I-D.ietf-ace-oauth-authz}} and proceeds according to Section 2.2 of {{I-D.seitz-ace-oscoap-profile}}.

The Group Manager stores the proof-of-possession key conveyed in the Access Token as the raw public key of the joining node.

Once the secure channel has been established, the joining node accesses the join resource hosted by the Group Manager, in order to join the associated OSCOAP multicast group. In particular, the joining node sends to the Group Manager a POST request targeting the join endpoint and specifying its intended role(s) in the multicast group. As defined in {{I-D.tiloca-core-multicast-oscoap}}, the joining node can be registered as multicaster and/or (pure) listener.

The Group Manager processes the request according to {{I-D.ietf-ace-oauth-authz}}. If this yields to a positive response, the Group Manager replies to the joining node with the following pieces of information:

* The endpoint ID, if the joining node is not configured exclusively as pure listener (Section 3 of {{I-D.tiloca-core-multicast-oscoap}}). The endpoint ID is at any time unique within a same multicast group.

* The Security Common Context associated to the joined multicast group (Section 4 of {{I-D.tiloca-core-multicast-oscoap}}).

* The public keys of the non-pure listeners currently in the joined multicast group, if the joining node is configured (also) as multicaster.

* The public keys of the multicasters currently in the joined multicast group, if the joining node is configured (also) as non-pure listener node.

From then on, the joining node is registered as a member of the multicast group, and can exchange group messages secured with OSCOAP as described in Section 6 of {{I-D.tiloca-core-multicast-oscoap}}.

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
