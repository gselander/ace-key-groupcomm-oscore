---
title: Key Management for OSCORE Groups in ACE
abbrev: Key Management for OSCORE Groups in ACE
docname: draft-ietf-ace-key-groupcomm-oscore-latest
# date: 2018-01-07
category: std

ipr: trust200902
area: Security
workgroup: ACE Working Group
keyword: Internet-Draft

# stand_alone: yes

coding: us-ascii

#pi:    # can use array (if all yes) or hash here
pi: [toc, sortrefs, symrefs]
#  - toc
#  - sortrefs
#  - symrefs
#toc: yes
#sortrefs:   # defaults to yes
#symrefs: yes

author:
-
    ins: M. Tiloca
    name: Marco Tiloca
    org: RISE AB
    street: Isafjordsgatan 22
    city: Kista
    code: SE-164 29 Stockholm
    country: Sweden
    email: marco.tiloca@ri.se
-
    ins: J. Park
    name: Jiye Park
    org: Universitaet Duisburg-Essen
    street: Schuetzenbahn 70
    city: Essen
    code: 45127
    country: Germany
    email: ji-ye.park@uni-due.de
-
    ins: F. Palombini
    name: Francesca Palombini
    org: Ericsson AB
    street: Torshamnsgatan 23
    city: Kista
    code: SE-16440 Stockholm
    country: Sweden
    email: francesca.palombini@ericsson.com

normative:
  RFC2119:
  RFC5705:
  RFC6838:
  RFC6979:
  RFC7252:
  RFC8017:
  RFC8032:
  RFC8126:
  RFC8174:
  RFC8446:
  RFC8447:
  RFC8610:
  RFC8613:
  I-D.bormann-core-ace-aif:
  I-D.ietf-cose-rfc8152bis-struct:
  I-D.ietf-cose-rfc8152bis-algs:
  I-D.ietf-core-oscore-groupcomm:
  I-D.ietf-ace-key-groupcomm:
  I-D.ietf-ace-oauth-authz:
  I-D.ietf-ace-oscore-profile:
  COSE.Algorithms:
    author:
      org: IANA
    date: false
    title: COSE Algorithms
    target: https://www.iana.org/assignments/cose/cose.xhtml#algorithms
  COSE.Key.Types:
    author:
      org: IANA
    date: false
    title: COSE Key Types
    target: https://www.iana.org/assignments/cose/cose.xhtml#key-type
  COSE.Elliptic.Curves:
    author:
      org: IANA
    date: false
    title: COSE Elliptic Curves
    target: https://www.iana.org/assignments/cose/cose.xhtml#elliptic-curves
  CWT.Confirmation.Methods:
    author:
      org: IANA
    date: false
    title: COSE Elliptic Curves
    target: https://www.iana.org/assignments/cwt/cwt.xhtml#confirmation-methods
  CORE.Parameters:
    author:
        org: IANA
    date: false
    title: Constrained RESTful Environments (CoRE) Parameters
    target: https://www.iana.org/assignments/core-parameters/core-parameters.xhtml
       
informative:
  I-D.ietf-core-groupcomm-bis:
  I-D.ietf-ace-dtls-authorize:
  I-D.ietf-core-coap-pubsub:
  I-D.tiloca-core-oscore-discovery:
  I-D.ietf-core-echo-request-tag:
  I-D.ietf-ace-dtls-authorize:
  I-D.ietf-ace-oscore-gm-admin:
  RFC6347:
  RFC6690:
  RFC6749:
  RFC7641:

--- abstract

This specification defines an application profile of the ACE framework for Authentication and Authorization, to request and provision keying material in group communication scenarios that are based on CoAP and secured with Group Object Security for Constrained RESTful Environments (OSCORE). This application profile delegates the authentication and authorization of Clients that join an OSCORE group through a Resource Server acting as Group Manager for that group. This application profile leverages protocol-specific transport profiles of ACE to achieve communication security, server authentication and proof-of-possession for a key owned by the Client and bound to an OAuth 2.0 Access Token.

--- middle

# Introduction {#sec-introduction}

Object Security for Constrained RESTful Environments (OSCORE) {{RFC8613}} is a method for application-layer protection of the Constrained Application Protocol (CoAP) {{RFC7252}}, using CBOR Object Signing and Encryption (COSE) {{I-D.ietf-cose-rfc8152bis-struct}}{{I-D.ietf-cose-rfc8152bis-algs}} and enabling end-to-end security of CoAP payload and options.

As described in {{I-D.ietf-core-oscore-groupcomm}}, Group OSCORE is used to protect CoAP group communication over IP multicast {{I-D.ietf-core-groupcomm-bis}}. This relies on a Group Manager, which is responsible for managing an OSCORE group and enables the group members to exchange CoAP messages secured with Group OSCORE. The Group Manager can be responsible for multiple groups, coordinates the joining process of new group members, and is entrusted with the distribution and renewal of group keying material.

This specification is an application profile of {{I-D.ietf-ace-key-groupcomm}}, which itself builds on the ACE framework for Authentication and Authorization {{I-D.ietf-ace-oauth-authz}}. Message exchanges among the participants as well as message formats and processing follow what specified in {{I-D.ietf-ace-key-groupcomm}} for provisioning and renewing keying material in group communication scenarios, where Group OSCORE is used to protect CoAP group communication over IP multicast.

## Terminology {#ssec-terminology}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}}{{RFC8174}} when, and only when, they appear in all capitals, as shown here.

Readers are expected to be familiar with:

* The terms and concepts described in the ACE framework for authentication and authorization {{I-D.ietf-ace-oauth-authz}} and in the Authorization Information Format (AIF) {{I-D.bormann-core-ace-aif}} to express authorization information. The terminology for entities in the considered architecture is defined in OAuth 2.0 {{RFC6749}}. In particular, this includes Client (C), Resource Server (RS), and Authorization Server (AS).

* The terms and concepts related to the CoAP protocol described in {{RFC7252}}{{I-D.ietf-core-groupcomm-bis}}. Unless otherwise indicated, the term "endpoint" is used here following its OAuth definition, aimed at denoting resources such as /token and /introspect at the AS and /authz-info at the RS. This document does not use the CoAP definition of "endpoint", which is "An entity participating in the CoAP protocol".

* The terms and concept related to the message formats and processing specified in {{I-D.ietf-ace-key-groupcomm}}, for provisioning and renewing keying material in group communication scenarios.

* The terms and concepts for protection and processing of CoAP messages through OSCORE {{RFC8613}} and through Group OSCORE {{I-D.ietf-core-oscore-groupcomm}} in group communication scenarios. These include the concept of Group Manager, as the entity responsible for a set of groups where communications are secured with Group OSCORE. In this specification, the Group Manager acts as Resource Server.

Additionally, this document makes use of the following terminology.

* Requester: member of an OSCORE group that sends request messages to other members of the group.

* Responder: member of an OSCORE group that receives request messages from other members of the group. A responder may reply back, by sending a response message to the requester which has sent the request message.

* Monitor: member of an OSCORE group that is configured as responder and never replies back to requesters after receiving request messages. This corresponds to the term "silent server" used in {{I-D.ietf-core-oscore-groupcomm}}.

* Signature verifier: entity external to the OSCORE group and intended to verify the countersignature of messages exchanged in the group. An authorized signature verifier does not join the OSCORE group as an actual member, yet it can retrieve the public keys of the current group members from the Group Manager.

# Protocol Overview {#sec-protocol-overview}

Group communication for CoAP over IP multicast has been enabled in {{I-D.ietf-core-groupcomm-bis}} and can be secured with Group Object Security for Constrained RESTful Environments (OSCORE) {{RFC8613}} as described in {{I-D.ietf-core-oscore-groupcomm}}. A network node joins an OSCORE group by interacting with the responsible Group Manager. Once registered in the group, the new node can securely exchange messages with other group members.

This specification describes how to use {{I-D.ietf-ace-key-groupcomm}} and {{I-D.ietf-ace-oauth-authz}} to perform a number of authentication, authorization and key distribution actions, as defined in Section 2 of {{I-D.ietf-ace-key-groupcomm}}, for an OSCORE group.

With reference to {{I-D.ietf-ace-key-groupcomm}}:

* The node wishing to join the OSCORE group, i.e. the joining node, is the Client.

* The Group Manager is the Key Distribution Center (KDC), acting as a Resource Server.

* The Authorization Server associated to the Group Manager is the AS.

All communications between the involved entities MUST be secured.

In particular, communications between the Client and the Group Manager leverage protocol-specific transport profiles of ACE to achieve communication security, proof-of-possession and server authentication. Note that it is expected that in the commonly referred base-case of this specification, the transport profile to use is pre-configured and well-known to nodes participating in constrained applications.

{{profile-req}} lists the specifications on this application profile of ACE, based on the requirements defined in Appendix A of {{I-D.ietf-ace-key-groupcomm}}.

## Overview of the Joining Process {#ssec-overview-join-process}

A node performs the steps described in Section 4.2 of {{I-D.ietf-ace-key-groupcomm}} in order to join an OSCORE group. The format and processing of messages exchanged among the participants are further specified in {{sec-joining-node-to-AS}} and {{sec-joining-node-to-GM}} of this document.

## Overview of the Group Rekeying Process {#ssec-overview-group-rekeying-process}

If the application requires backward and forward security, the Group Manager MUST generate new keying material and distribute it to the group (rekeying) upon membership changes.

That is, the group is rekeyed when a node joins the group as a new member, or after a current member leaves the group. By doing so, a joining node cannot access communications in the group prior its joining, while a leaving node cannot access communications in the group after its leaving.

The keying material distributed through a group rekeying MUST include:

* a new Group Identifier (Gid) for the group as introduced in {{I-D.ietf-ace-key-groupcomm}}, used as ID Context parameter of the OSCORE Common Security Context of that group (see Section 2 of {{I-D.ietf-core-oscore-groupcomm}}).

   Note that the Gid differs from the plain group name also introduced in {{I-D.ietf-ace-key-groupcomm}}, which is a plain, stable and invariant identifier, with no cryptographic relevance and meaning.

* a new value for the Master Secret parameter of the OSCORE Common Security Context of that group (see Section 2 of {{I-D.ietf-core-oscore-groupcomm}}).

Also, the distributed keying material MAY include a new value for the Master Salt parameter of the OSCORE Common Security Context of that group.

Upon generating the new group keying material and before starting its distribution, the Group Manager MUST increment the version number of the group keying material. When rekeying a group, the Group Manager MUST preserve the current value of the Sender ID of each member in that group.

The Group Manager MUST support the Group Rekeying Process described in {{sec-group-rekeying-process}}. Future application profiles may define alternative message formats and distribution schemes to perform group rekeying.

# Format of Scope {#sec-format-scope}

Building on Section 3.1 of {{I-D.ietf-ace-key-groupcomm}}, this section defines the exact format and encoding of scope to use.

To this end, this profile uses the Authorization Information Format (AIF) {{I-D.bormann-core-ace-aif}}, and defines the following AIF specific data model AIF-OSCORE-GROUPCOMM.

With reference to the generic AIF model

~~~~~~~~~~~
   AIF-Generic<Toid, Tperm> = [* [Toid, Tperm]]
~~~~~~~~~~~

the value of the CBOR byte string used as scope encodes the CBOR array \[* \[Toid, Tperm\]\], where each \[Toid, Tperm\] element corresponds to one scope entry.

Then, for each scope entry:

* the object identifier ("Toid") is specialized as a CBOR text string, specifying the group name for the scope entry;

* the permission set ("Tperm") is specialized to a CBOR unsigned integer with value R, specifying the role(s) that the client wishes to take in the group (REQ2). The value R is computed as follows:

   - each role in the permission set is converted into the corresponding numeric identifier X from the "Value" column of the table in {{fig-role-values}}.

   - the set of N numbers is converted into the single value R, by taking each numeric identifier X_1, X_2, ..., X_N to the power of two, and then computing the inclusive OR of the binary representations of all the power values.

~~~~~~~~~~~
+-----------+-------+-------------------------------------------------+
| Name      | Value | Description                                     |
+===========+=======+=================================================+
| Reserved  | 0     | This value is reserved                          |
|-----------+-------+-------------------------------------------------+
| Requester | 1     | Send requests; receive responses                |
|-----------+-------+-------------------------------------------------+
| Responder | 2     | Send responses; receive requests                |
+-----------+-------+-------------------------------------------------+
| Monitor   | 3     | Receive requests; never send requests/responses |
|-----------+-------+-------------------------------------------------|
| Verifier  | 4     | Verify countersignature of intercepted messages |
+-----------+-------+-------------------------------------------------+
~~~~~~~~~~~
{: #fig-role-values title="Numeric identifier of roles in the OSCORE group" artwork-align="center"}

The CDDL {{RFC8610}} definition of the AIF-OSCORE-GROUPCOMM data model is as follows:

~~~~~~~~~~~
   AIF-OSCORE-GROUPCOMM = AIF_Generic<path, permissions>

   path = tstr  ; Group name
   permissions = uint . bits roles
   roles = &(
      Requester: 1,
      Responder: 2,
      Monitor: 3,
      Verifier: 4
   )
~~~~~~~~~~~

Future specifications that define new roles MUST register a corresponding numeric identifier in the "Group OSCORE Roles" Registry defined in {{ssec-iana-group-oscore-roles-registry}} of this specification.

# Joining Node to Authorization Server {#sec-joining-node-to-AS}

This section describes how the joining node interacts with the AS in order to be authorized to join an OSCORE group under a given Group Manager. In particular, it considers a joining node that intends to contact that Group Manager for the first time.

The message exchange between the joining node and the AS consists of the messages Authorization Request and Authorization Response defined in Section 3 of {{I-D.ietf-ace-key-groupcomm}}. Note that what is defined in {{I-D.ietf-ace-key-groupcomm}} applies, and only additions or modifications to that specification are defined here.

## Authorization Request {#ssec-auth-req}

The Authorization Request message is as defined in Section 3.1 of {{I-D.ietf-ace-key-groupcomm}}, with the following additions.

* If the 'scope' parameter is present:

   - The value of the CBOR byte string encodes a CBOR array, whose format MUST follow the data model AIF-OSCORE-GROUPCOMM defined in {{sec-format-scope}}. In particular, for each OSCORE group to join:

      - The group name is encoded as a CBOR text string.

      - The set of requested roles is expressed as a single CBOR unsigned integer, computed as defined in {{sec-format-scope}} (REQ2) from the numerical abbreviations defined in {{fig-role-values}} for each requested role (OPT7).

## Authorization Response {#ssec-auth-resp}

The Authorization Response message is as defined in Section 3.2 of {{I-D.ietf-ace-key-groupcomm}}, with the following additions:

* The AS MUST include the 'expires_in' parameter. Other means for the AS to specify the lifetime of Access Tokens are out of the scope of this specification.

* The AS MUST include the 'scope' parameter, when the value included in the Access Token differs from the one specified by the joining node in the request. In such a case, the second element of each scope entry MUST be present, and specifies the set of roles that the joining node is actually authorized to take in the OSCORE group for that scope entry, encoded as specified in {{ssec-auth-req}}.

# Interface at the Group Manager {#sec-interface-GM}

The Group Manager provides the interface defined in Section 4.1 of {{I-D.ietf-ace-key-groupcomm}}, with the following additional resource:

* /ace-group/GROUPNAME/active: this sub-resource supports the GET method, whose handler is defined in {{active-get}}.

The GROUPNAME segment of the URI path and the group name specified in the Access Token scope (gname in Section 3.1 of {{I-D.ietf-ace-key-groupcomm}}) MUST match (REQ1).

The Resource Type (rt=) Link Target Attribute value "core.osc.gm" is registered in {{iana-rt}} (REQ7a), and can be used to describe group-membership resources and its sub-resources at a Group Manager, e.g. using a link-format document {{RFC6690}}.

Applications can use this common resource type to discover links to group-membership resources for joining OSCORE groups, e.g. by using the approach described in {{I-D.tiloca-core-oscore-discovery}}.

## GET Handler {#active-get}

The handler expects a GET request.

The handler verifies that the group identifier of the /ace-group/GROUPNAME/active path is a subset of the 'scope' stored in the Access Token associated to the requesting client. If verification fails, the Group Manager MUST respond with a 4.01 (Unauthorized) error message.

If verification succeeds, the handler returns a 2.05 (Content) message containing the CBOR simple value True if the group is currently active, or the CBOR simple value False otherwise. The group is considered active if it is set to allow new members to join, and if communication within the group is expected.

The method to set the current group status, i.e. active or inactive, is out of the scope of this specification, and is defined for the administrator interface of the Group Manager specified in {{I-D.ietf-ace-oscore-gm-admin}}.

# Token POST and Group Joining {#sec-joining-node-to-GM}

The following subsections describe the interactions between the joining node and the Group Manager, i.e. the sending of the Access Token and the Request-Response exchange to join the OSCORE group. The message exchange between the joining node and the KDC consists of the messages defined in Section 3.3 and 4.2 of {{I-D.ietf-ace-key-groupcomm}}. Note that what is defined in {{I-D.ietf-ace-key-groupcomm}} applies, and only additions or modifications to that specification are defined here.

A signature verifier provides the Group Manager with an Access Token, as described in {{ssec-token-post}}, just as any another joining node does. However, unlike candidate group members, it does not join any OSCORE group, i.e. it does not perform the joining process defined in {{ssec-join-req-sending}}. After successfully posting an Access Token, a signature verifier is authorized to perform only the operations specified in {{sec-pub-keys}}, to retrieve the public keys of group members, and only for the OSCORE groups specified in the validated Access Token. The Group Manager MUST respond with a 4.01 (Unauthorized) error message, in case a signature verifier attempts to access any other endpoint than /ace-group/GROUPNAME/pub-key at the Group Manager.

## Token Post {#ssec-token-post}

The Token post exchange is defined in Section 3.3 of {{I-D.ietf-ace-key-groupcomm}}.

Additionally to what defined in {{I-D.ietf-ace-key-groupcomm}}, the following applies.

* The 'kdcchallenge' parameter contains a dedicated nonce N_S generated by the Group Manager. For the N\_S value, it is RECOMMENDED to use a 8-byte long random nonce. The joining node may use this nonce in order to prove the possession of its own private key, upon joining the group (see {{ssec-join-req-sending}}).

    The 'kdcchallenge' parameter MAY be omitted from the 2.01 (Created) response, if the 'scope' of the Access Token specifies only the role "monitor" or only the role "verifier" or both of them, for each and every of the specified groups.

* If the 'sign_info' parameter is present in the response, the following applies for each element 'sign_info_entry'.

  * 'sign_alg' takes value from the "Value" column of the "COSE Algorithms" Registry {{COSE.Algorithms}}.

  * 'sign_parameters' is a CBOR array including the following two elements:

     - 'sign_alg_capab', encoded as a CBOR array. Its precise format and value is the same as the COSE capabilities entry in the "Capabilities" column of the "COSE Algorithms" Registry {{COSE.Algorithms}}, for the algorithm indicated in 'sign_alg' (REQ4).

     - 'sign_key_type_capab', encoded as a CBOR array.  Its precise format and value is the same as the COSE capabilities entry in the "Capabilities" column of the "COSE Key Types" Registry {{COSE.Key.Types}}, for the algorithm indicated in 'sign_alg' (REQ4).

  * 'sign_key_parameters' is a CBOR array.  Its precise format and value is the same as the COSE capabilities entry in the "Capabilities" column of the "COSE Key Types" Registry {{COSE.Key.Types}}, for the algorithm indicated in 'sign_alg' (REQ5).

  * 'pub_key_enc' takes value 1 ("COSE\_Key") from the 'Confirmation Key' column of the "CWT Confirmation Method" Registry {{CWT.Confirmation.Methods}}, so indicating that public keys in the OSCORE group are encoded as COSE Keys {{I-D.ietf-cose-rfc8152bis-struct}}. Future specifications may define additional values for this parameter.

Note that, other than through the above parameters as defined in Section 3.3 of {{I-D.ietf-ace-key-groupcomm}}, the joining node MAY have previously retrieved this information by other means, e.g. by using the approach described in {{I-D.tiloca-core-oscore-discovery}} to discover the OSCORE group and the link to the associated group-membership resource at the Group Manager.

Additionally, if allowed by the used transport profile of ACE, the joining node may instead provide the Access Token to the Group Manager by other means, e.g. during a secure session establishment (see Section 3.3.1 of {{I-D.ietf-ace-dtls-authorize}}).

## Sending the Joining Request {#ssec-join-req-sending}

The joining node requests to join the OSCORE group, by sending a Joining Request message to the related group-membership resource at the Group Manager, as per Section 4.2 of {{I-D.ietf-ace-key-groupcomm}}.

Additionally to what defined in {{I-D.ietf-ace-key-groupcomm}}, the following applies.

* The 'scope' parameter MUST be included. Its value encodes one scope entry with the format defined in {{sec-format-scope}}, indicating the group name and the role(s) that the joining node wants to take in the group.

* The 'get_pub_keys' parameter is present only if the joining node wants to retrieve the public keys of the group members from the Group Manager during the joining process (see {{sec-public-keys-of-joining-nodes}}). Otherwise, this parameter MUST NOT be present.

   If this parameter is present, each element (if any) of the first CBOR array is encoded as a CBOR integer, with the same value of a permission set ("Tperm") indicating that role or combination of roles in a scope entry, as defined in {{sec-format-scope}}.

* 'cnonce' contains a dedicated nonce N_C generated by the joining node. For the N\_C value, it is RECOMMENDED to use a 8-byte long random nonce.

* The signature encoded in the 'client_cred_verify' parameter is computed by the joining node by using the same private key and countersignature algorithm it intends to use for signing messages in the OSCORE group. Moreover, N_S is as defined in {{sssec-challenge-value}}.

### Value of the N\_S Challenge {#sssec-challenge-value}

The value of the N\_S challenge is determined as follows.

1. If the joining node has posted the Access Token to the /authz-info endpoint of the Group Manager as in {{ssec-token-post}}, N\_S takes the same value of the most recent 'kdcchallenge' parameter received by the joining node from the Group Manager. This can be either the one specified in the 2.01 (Created) response to the Token POST, or the one possibly specified in a 4.00 (Bad Request) response to a following Joining Request (see {{ssec-join-req-processing}}).

2. If the Token posting has relied on the DTLS profile of ACE {{I-D.ietf-ace-dtls-authorize}} with the Access Token as content of the "psk_identity" field of the ClientKeyExchange message {{RFC6347}}, N\_S is an exporter value computed as defined in Section 7.5 of {{RFC8446}}. Specifically, N\_S is exported from the DTLS session between the joining node and the Group Manager, using an empty 'context_value', 32 bytes as 'key_length', and the exporter label "EXPORTER-ACE-Sign-Challenge-coap-group-oscore-app" defined in {{ssec-iana-tls-esporter-label-registry}} of this specification.

It is up to applications to define how N_S is computed in further alternative settings.

{{ssec-security-considerations-reusage-nonces}} provides security considerations on the reusage of the N_S challenge.

## Processing the Joining Request {#ssec-join-req-processing}

The Group Manager processes the Joining Request as defined in Section 4.1.2.1 of {{I-D.ietf-ace-key-groupcomm}}. Additionally, the following applies.

* In case the Joining Request does not include the 'client_cred' parameter, the joining process fails if the Group Manager either: i) does not store a public key with an accepted format for the joining node; or ii) stores multiple public keys with an accepted format for the joining node.

* To compute the signature contained in 'client_cred_verify', the GM considers:*

   - as signed value, the value of the 'scope' parameter from the Joining Request concatenated with N_S concatenated with N_C, where N_S is determined as described in {{sssec-challenge-value}}, while N_C is the nonce provided in the 'cnonce' parameter of the Joining Request;
   
   - the countersignature algorithm used in the OSCORE group, and possible correponding parameters;
   
   - the public key of the joining node, either retrieved from the 'client_cred' parameter, or already stored as acquired from previous interactions with the joining node.

* A 4.00 Bad Request response from the Group Manager to the joining node MUST have content format application/ace+cbor. The response payload is a CBOR map which MUST contain the 'sign_info' parameter, including a single element 'sign_info_entry' pertaining to the OSCORE group that the joining node tried to join with the Joining Request.

* The Group Manager MUST return a 4.00 (Bad Request) response in case the 'scope' parameter is not present in the Joining Request, or if it is present and specifies any set of roles not included in the following list: "requester", "responder", "monitor", ("requester", "responder"). Future specifications that define a new role MUST define possible sets of roles including the new one and existing ones, that are acceptable to specify in the 'scope' parameter of a Joining Request.

* The Group Manager MUST return a 4.00 (Bad Request) response in case the Joining Request includes the 'client_cred' parameter but does not include both the 'cnonce' and 'client_cred_verify' parameters.

* The Group Manager MUST return a 4.00 (Bad Request) response in case it cannot retrieve a public key with an accepted format for the joining node, either from the 'client_cred' parameter or as already stored.

* When receiving a 4.00 Bad Request response, the joining node SHOULD send a new Joining Request to the Group Manager, where:

  * The 'cnonce' parameter MUST include a new dedicated nonce N\_C generated by the joining node.

  * The 'client_cred' parameter MUST include a public key compatible with the encoding, countersignature algorithm and possible associated parameters indicated by the Group Manager.

  * The 'client_cred_verify' parameter MUST include a signature computed as described in {{ssec-join-req-sending}}, by using the public key indicated in the current 'client_cred' parameter, with the countersignature algorithm and possible associated parameters indicated by the Group Manager. If the error response from the Group Manager included the 'kdcchallenge' parameter, the joining node MUST use its content as new N\_S challenge to compute the signature.

## Joining Response {#ssec-join-resp}

If the processing of the Joining Request described in {{ssec-join-req-processing}} is successful, the Group Manager updates the group membership by registering the joining node NODENAME as a new member of the OSCORE group GROUPNAME, as described in Section 4.1.2.1 of {{I-D.ietf-ace-key-groupcomm}}.

If the joining node has not taken exclusively the role of monitor, the Group Manager performs also the following actions.

* The Group Manager selects an available OSCORE Sender ID in the OSCORE group, and exclusively assigns it to the joining node.

* The Group Manager stores the association between i) the public key of the joining node; and ii) the Group Identifier (Gid), i.e. the OSCORE ID Context, associated to the OSCORE group together with the OSCORE Sender ID assigned to the joining node in the group. The Group Manager MUST keep this association updated over time.

Then, the Group Manager replies to the joining node, providing the updated security parameters and keying meterial necessary to participate in the group communication. This success Joining Response is formatted as defined in Section 4.1.2.1 of {{I-D.ietf-ace-key-groupcomm}}, with the following additions:

* The 'gkty' parameter identifies a key of type "Group_OSCORE_Security_Context object", defined in {{ssec-iana-groupcomm-key-registry}} of this specification.

* The 'key' parameter includes what the joining node needs in order to set up the OSCORE Security Context as per Section 2 of {{I-D.ietf-core-oscore-groupcomm}}. This parameter has as value a Group_OSCORE_Security_Context object, which is defined in this specification and extends the OSCORE_Security_Context object encoded in CBOR as defined in Section 3.2.1 of {{I-D.ietf-ace-oscore-profile}}. In particular, it contains the additional parameters 'cs_alg', 'cs_params', 'cs_key_params' and 'cs_key_enc' defined in {{ssec-iana-security-context-parameter-registry}} of this specification. More specifically, the 'key' parameter is composed as follows.

   * The 'ms' parameter MUST be present and includes the OSCORE Master Secret value.

   * The 'clientId' parameter, if present, has as value the OSCORE Sender ID assigned to the joining node by the Group Manager, as described above. This parameter is not present if the node joins the group exclusively with the role of monitor, according to what specified in the Access Token (see {{ssec-auth-resp}}). In any other case, this parameter MUST be present.
   
      Note that this parameter and its value have no relation with the CoAP role that the joining node is going to take in the group. That is, it is not indicative of whether the joining node is going to act (also) as CoAP client once in the group.

   * The 'hkdf' parameter, if present, has as value the KDF algorithm used in the group.

   * The 'alg' parameter, if present, has as value the AEAD algorithm used in the group.

   * The 'salt' parameter, if present, has as value the OSCORE Master Salt.

   * The 'contextId' parameter MUST be present and has as value the Group Identifier (Gid), i.e. the OSCORE ID Context of the OSCORE group.

   * The 'cs_alg' parameter MUST be present and specifies the algorithm used to countersign messages in the group. This parameter takes values from the "Value" column of the "COSE Algorithms" Registry {{COSE.Algorithms}}.

   * The 'cs_params' parameter MAY be present and specifies the parameters for the counter signature algorithm. This parameter is a CBOR array, which includes the following two elements:

     - 'sign_alg_capab', with the same encoding as defined in {{ssec-token-post}}. The value is the same as in the Token Post response where the 'sign_parameters' value was non-null.

     - 'sign_key_type_capab', with the same encoding as defined in {{ssec-token-post}}. The value is the same as in the Token Post response where the 'sign_parameters' value was non-null.

   * The 'cs_key_params' parameter MAY be present and specifies the parameters for the key used with the counter signature algorithm. This parameter is a CBOR array, with the same non-null encoding and value as 'sign_key_parameters' of the {{ssec-token-post}}.

  * The 'cs_key_enc' parameter MAY be present and specifies the encoding of the public keys of the group members. This parameter is a CBOR integer, whose value is 1 ("COSE\_Key") taken from the 'Confirmation Key' column of the "CWT Confirmation Method" Registry {{CWT.Confirmation.Methods}}, so indicating that public keys in the OSCORE group are encoded as COSE Keys {{I-D.ietf-cose-rfc8152bis-struct}}. Future specifications may define additional values for this parameter. If this parameter is not present, 1 ("COSE\_Key") MUST be assumed as default value.

* The 'exp' parameter MUST be present.

* The 'ace-groupcomm-profile' parameter MUST be present and has value coap_group_oscore_app (TBD1), which is defined in {{ssec-iana-groupcomm-profile-registry}} of this specification.

* The 'pub_keys' parameter, if present, includes the public keys of the group members that are relevant to the joining node. That is, it includes: i) the public keys of the responders currently in the group, in case the joining node is configured (also) as requester; and ii) the public keys of the requesters currently in the group, in case the joining node is configured (also) as responder or monitor. If public keys are encoded as COSE\_Keys, each of them has as 'kid' the Sender ID that the corresponding owner has in the group, thus used as group member identifier.

* The 'group_policies' parameter SHOULD be present, and SHOULD include the following elements:

   * "Sequence Number Synchronization Method" defined in Section 4.1.2.1 of {{I-D.ietf-ace-key-groupcomm}}, with default value 1 ("Best effort");

   * "Key Update Check Interval" defined in Section 4.1.2.1 of {{I-D.ietf-ace-key-groupcomm}}, with default value 3600;

   * "Expiration Delta" defined in Section 4.1.2.1 of {{I-D.ietf-ace-key-groupcomm}}, with default value 0.

   * "Group OSCORE Pairwise Mode Support" defined in {{ssec-pairwise-mode-policy}} of this specification, with default value False.

Finally, the joining node uses the information received in the Joining Response to set up the OSCORE Security Context, as described in Section 2 of {{I-D.ietf-core-oscore-groupcomm}}. In addition, the joining node maintains an association between each public key retrieved from the 'pub_keys' parameter and the role(s) that the corresponding group member has in the group.

From then on, the joining node can exchange group messages secured with Group OSCORE as described in {{I-D.ietf-core-oscore-groupcomm}}. When doing so:

* The joining node MUST NOT process an incoming request message, if signed by a group member whose public key is not associated to the role "Requester".

* The joining node MUST NOT process an incoming response message, if signed by a group member whose public key is not associated to the role "Responder".

If the application requires backward security, the Group Manager MUST generate updated security parameters and group keying material, and provide it to the current group members upon the new node's joining (see {{sec-group-rekeying-process}}). As a consequence, the joining node is not able to access secure communication in the group occurred prior its joining.

## ACE Groupcomm Policy for Group OSCORE Pairwise Mode Support ## {#ssec-pairwise-mode-policy}

This specifications defines the group policy "Group OSCORE Pairwise Mode Support", for which it registers an entry in the "ACE Groupcomm Policy" IANA Registry defined in Section 8.8 of {{I-D.ietf-ace-key-groupcomm}}.

The corresponding element in the 'group_policies' parameter of the Joining Response (see {{ssec-join-resp}}) encodes the CBOR simple value True, if the OSCORE group supports the pairwise mode of Group OSCORE {{I-D.ietf-core-oscore-groupcomm}}, or the CBOR simple value False otherwise (REQ14).

# Public Keys of Joining Nodes # {#sec-public-keys-of-joining-nodes}

Source authentication of a message sent within the group and protected with Group OSCORE is ensured by means of a digital counter signature embedded in the message (in group mode), or by integrity-protecting the message with pairwise keying material derived from the asymmetric keys of sender and recipient (in pairwise mode).

Therefore, group members must be able to retrieve each other's public key from a trusted key repository, in order to verify source authenticity of incoming group messages.

As also discussed in {{I-D.ietf-core-oscore-groupcomm}}, the Group Manager acts as trusted repository of the public keys of the group members, and provides those public keys to group members if requested to. Upon joining an OSCORE group, a joining node is thus expected to provide its own public key to the Group Manager.

In particular, one of the following four cases can occur when a new node joins an OSCORE group.

* The joining node is going to join the group exclusively as monitor. That is, it is not going to send messages to the group, and hence to produce signatures with its own private key. In this case, the joining node is not required to provide its own public key to the Group Manager, which thus does not have to perform any check related to the public key encoding, or to a countersignature algorithm and possible associated parameters for that joining node. In case that joining node still provides a public key in the 'client_cred' parameter of the Joining Request (see {{ssec-join-req-sending}}), the Group Manager silently ignores that parameter, as well as related the parameters 'cnonce' and 'client_cred_verify'.

* The Group Manager already acquired the public key of the joining node during a past joining process. In this case, the joining node MAY choose not to provide again its own public key to the Group Manager, in order to limit the size of the Joining Request. The joining node MUST provide its own public key again if it has provided the Group Manager with multiple public keys during past joining processes, intended for different OSCORE groups. If the joining node provides its own public key, the Group Manager performs consistency checks as per {{ssec-join-req-processing}} and, in case of success, considers it as the public key associated to the joining node in the OSCORE group.

* The joining node and the Group Manager use an asymmetric proof-of-possession key to establish a secure communication channel. Then, two cases can occur.

   1. The proof-of-possession key is compatible with the encoding as well as with the counter signature algorithm and possible associated parameters used in the OSCORE group. Then, the Group Manager considers the proof-of-possession key as the public key associated to the joining node in the OSCORE group. If the joining node is aware that the proof-of-possession key is also valid for the OSCORE group, it MAY not provide it again as its own public key to the Group Manager. The joining node MUST provide its own public key again if it has provided the Group Manager with multiple public keys during past joining processes, intended for different OSCORE groups. If the joining node provides its own public key in the 'client_cred' parameter of the Joining Request (see {{ssec-join-req-sending}}), the Group Manager performs consistency checks as per {{ssec-join-req-processing}} and, in case of success, considers it as the public key associated to the joining node in the OSCORE group.

   2. The proof-of-possession key is not compatible with the encoding or with the counter signature algorithm and possible associated parameters used in the OSCORE group. In this case, the joining node MUST provide a different compatible public key to the Group Manager in the 'client_cred' parameter of the Joining Request (see {{ssec-join-req-sending}}). Then, the Group Manager performs consistency checks on this latest provided public key as per {{ssec-join-req-processing}} and, in case of success, considers it as the public key associated to the joining node in the OSCORE group.

* The joining node and the Group Manager use a symmetric proof-of-possession key to establish a secure communication channel. In this case, upon performing a joining process with that Group Manager for the first time, the joining node specifies its own public key in the 'client_cred' parameter of the Joining Request targeting the group-membership endpoint (see {{ssec-join-req-sending}}).

# Retrieval of Updated Keying Material # {#sec-updated-key}

At some point, a group member considers the OSCORE Security Context invalid and to be renewed. This happens, for instance, after a number of unsuccessful security processing of incoming messages from other group members, or when the Security Context expires as specified by the 'exp' parameter of the Joining Response.

When this happens, the group member retrieves updated security parameters and group keying material. This can occur in the two different ways described below.

## Retrieval of Group Keying Material ## {#ssec-updated-key-only}

If the group member wants to retrieve only the latest group keying material, it sends a Key Distribution Request to the Group Manager.

In particular, it sends a CoAP GET request to the endpoint /ace-group/GROUPNAME at the Group Manager.

The Group Manager processes the Key Distribution Request according to Section 4.1.2.2 of {{I-D.ietf-ace-key-groupcomm}}. The Key Distribution Response is formatted as defined in Section 4.1.2.2 of {{I-D.ietf-ace-key-groupcomm}}. In particular, the 'key' parameter is formatted as defined in {{ssec-join-resp}} of this specification, with the difference that it does not include the 'clientId' parameter.

Upon receiving the Key Distribution Response, the group member retrieves the updated security parameters and group keying material, and, if they differ from the current ones, use them to set up the new OSCORE Security Context as described in Section 2 of {{I-D.ietf-core-oscore-groupcomm}}.

## Retrieval of Group Keying Material and Sender ID ## {#ssec-updated-and-individual-key}

If the group member wants to retrieve the latest group keying material as well as the Sender ID that it has in the OSCORE group, it sends a Key Distribution Request to the Group Manager.

In particular, it sends a CoAP GET request to the endpoint /ace-group/GROUPNAME/nodes/NODENAME at the Group Manager.

The Group Manager processes the Key Distribution Request according to Section 4.1.6.2 of {{I-D.ietf-ace-key-groupcomm}}. The Key Distribution Response is formatted as defined in Section 4.1.6.2 of {{I-D.ietf-ace-key-groupcomm}}.

In particular, the 'key' parameter is formatted as defined in {{ssec-join-resp}} of this specification, with the difference that if the requesting group member has exclusively the role of monitor, no 'clientId' is specified within the 'key' parameter. Note that, in any other case, the current Sender ID of the group member is not specified as a separate parameter, but rather specified as 'clientId' within the 'key' parameter.

Upon receiving the Key Distribution Response, the group member retrieves the updated security parameters, group keying material and Sender ID, and, if they differ from the current ones, use them to set up the new OSCORE Security Context as described in Section 2 of {{I-D.ietf-core-oscore-groupcomm}}.

# Requesting a Change of Keying Material # {#sec-new-key}

As discussed in Section 2.4.2 of {{I-D.ietf-core-oscore-groupcomm}}, a group member may at some point exhaust its Sender Sequence Numbers in the group.

When this happens, the group member MUST send a Key Renewal Request message to the Group Manager, as per Section 4.4 of {{I-D.ietf-ace-key-groupcomm}}. In particular, it sends a CoAP PUT request to the endpoint /ace-group/GROUPNAME/nodes/NODENAME at the Group Manager.

Upon receiving the Key Renewal Request, the Group Manager processes it as defined in Section 4.1.6.1 of {{I-D.ietf-ace-key-groupcomm}}, and performs one of the following actions.

1. If the requesting group member has exclusively the role of monitor, the Group Manager replies with a 4.00 (Bad Request) error response.

2. Otherwise, the Group Manager takes one of the following actions.

    a. The Group Manager rekeys the OSCORE group. That is, the Group Manager generates new group keying material for that group (see {{sec-group-rekeying-process}}), and replies to the group member with a group rekeying message as defined in {{sec-group-rekeying-process}}, providing the new group keying material. Then, the Group Manager rekeys the rest of the OSCORE group, as discussed in {{sec-group-rekeying-process}}.
    
    The Group Manager SHOULD perform a group rekeying only if already scheduled to  occur shortly, e.g. according to an application-dependent rekeying period, or as a reaction to a recent change in the group membership. In any other case, the Group Manager SHOULD NOT rekey the OSCORE group when receiving a Key Renewal Request (OPT8).

    b. The Group Manager generates a new Sender ID for that group member and replies with a Key Renewal Response, formatted as defined in Section 4.1.6.1 of {{I-D.ietf-ace-key-groupcomm}}. In particular, the CBOR Map in the response payload includes a single parameter 'clientId' defined in {{ssec-iana-ace-groupcomm-parameters-registry}} of this document, specifying the new Sender ID of the group member encoded as a CBOR byte string.

# Retrieval of Public Keys and Roles for Group Members # {#sec-pub-keys}

A group member or a signature verifier may need to retrieve the public keys of (other) group members. To this end, the group member or signature verifier sends a Public Key Request message to the Group Manager, as per Section 4.5 of {{I-D.ietf-ace-key-groupcomm}}. In particular, it sends the request to the endpoint /ace-group/GROUPNAME/pub-key at the Group Manager.

If the Public Key Request uses the method FETCH, the Public Key Request is formatted as defined in Section 4.1.3.1 of {{I-D.ietf-ace-key-groupcomm}}. In particular:

* Each element (if any) of the first CBOR array is formatted as in the first CBOR array of the 'get_pub_keys' parameter of the Joining Request (see {{ssec-join-req-sending}}).

* Each element (if any) of the second CBOR array is a CBOR byte string, which encodes the Sender ID of the group member for which the associated public key is requested.

Upon receiving the Public Key Request, the Group Manager processes it as per Section 4.1.3.1 or 4.1.3.2 of {{I-D.ietf-ace-key-groupcomm}}, depending on the request method being FETCH or GET, respectively. Additionally, if the Public Key Request uses the method FETCH, the Group Manager silently ignores identifiers included in the ’get_pub_keys’ parameter of the request that are not associated to any current group member.

The success Public Key Response is formatted as defined in Section 4.1.3.1 or 4.1.3.2 of {{I-D.ietf-ace-key-groupcomm}}, depending on the request method being FETCH or GET, respectively.

# Update of Public Key # {#sec-update-pub-key}

A group member may need to provide the Group Manager with its new public key to use in the group from then on, hence replacing the current one. This can be the case, for instance, if the countersignature algorithm and possible associated parameters used in the OSCORE group have been changed, and the current public key is not compatible with them.

To this end, the group member sends a Public Key Update Request message to the Group Manager, as per Section 4.6 of {{I-D.ietf-ace-key-groupcomm}}. In particular, it sends a CoAP POST request to the endpoint /ace-group/GROUPNAME/nodes/NODENAME/pub-key at the Group Manager.

Upon receiving the Group Leaving Request, the Group Manager processes it as per Section 4.1.7.1 of {{I-D.ietf-ace-key-groupcomm}}, with the following additions.

* If the requesting group member has exclusively the role of monitor, the Group Manager replies with a 4.00 (Bad request) error response.

* The N\_S signature challenge is computed as per point (3) in {{sssec-challenge-value}} (REQ17).

* If the request is successfully processed, the Group Manager stores the association between i) the new public key of the group member; and ii) the Group Identifier (Gid), i.e. the OSCORE ID Context, associated to the OSCORE group together with the OSCORE Sender ID assigned to the group member in the group. The Group Manager MUST keep this association updated over time.

# Retrieval of Group Policies # {#sec-policies}

A group member may request the current policies used in the OSCORE group. To this end, the group member sends a Policies Request, as per Section 4.7  of {{I-D.ietf-ace-key-groupcomm}}. In particular, it sends a CoAP GET request to the endpoint /ace-group/GROUPNAME/policies at the Group Manager, where GROUPNAME is the name of the OSCORE group.

Upon receiving the Policies Request, the Group Manager processes it as per Section 4.1.4.1 of {{I-D.ietf-ace-key-groupcomm}}. The success Policies Response is formatted as defined in Section 4.1.4.1 of {{I-D.ietf-ace-key-groupcomm}}.

# Retrieval of Keying Material Version # {#sec-version}

A group member may request the current version of the keying material used in the OSCORE group. To this end, the group member sends a Version Request, as per Section 4.8 of {{I-D.ietf-ace-key-groupcomm}}. In particular, it sends a CoAP GET request to the endpoint /ace-group/GROUPNAME/num at the Group Manager, where GROUPNAME is the name of the OSCORE group.

Upon receiving the Version Request, the Group Manager processes it as per Section 4.1.5.1 of {{I-D.ietf-ace-key-groupcomm}}. The success Version Response is formatted as defined in Section 4.1.5.1 of {{I-D.ietf-ace-key-groupcomm}}.

# Retrieval of Group Status # {#sec-status}

A group member may request the current status of the the OSCORE group, i.e. active or inactive. To this end, the group member sends a Group Status Request to the Group Manager.

In particular, the group member sends a CoAP GET request to the endpoint /ace-group/GROUPNAME/active at the Group Manager defined in {{sec-interface-GM}} of this specification, where GROUPNAME is the name of the OSCORE group. The success Group Version Response is formatted as defined in {{sec-interface-GM}} of this specification.

Upon learning from a 2.05 (Content) response that the group is currently inactive, the group member SHOULD stop taking part in communications within the group, until it becomes active again.

Upon learning from a 2.05 (Content) response that the group has become active again, the group member can resume taking part in communications within the group.

{{fig-key-status-req-resp}} gives an overview of the exchange described above.

~~~~~~~~~~~
 Group                                                         Group
 Member                                                       Manager
   |                                                             |
   |------ Group Status Request: GET ace-group/GID/active ------>|
   |                                                             |
   |<---------- Group Status Response: 2.05 (Content) -----------|
   |                                                             |
~~~~~~~~~~~
{: #fig-key-status-req-resp title="Message Flow of Group Status Request-Response" artwork-align="center"}

# Request to Leave the Group # {#sec-leave-req}

A group member may request to leave the OSCORE group. To this end, the group member sends a Group Leaving Request, as per Section 4.9 of {{I-D.ietf-ace-key-groupcomm}}. In particular, it sends a CoAP DELETE request to the endpoint /ace-group/GROUPNAME/nodes/NODENAME at the Group Manager.

Upon receiving the Group Leaving Request, the Group Manager processes it as per Section 4.1.6.3 of {{I-D.ietf-ace-key-groupcomm}}.

# Removal of a Group Member # {#sec-leaving}

Other than after a spontaneous request to the Group Manager as described in {{sec-leave-req}}, a node may be forcibly removed from the OSCORE group, e.g. due to expired or revoked authorization.

If, upon joining the group (see {{ssec-join-req-sending}}), the leaving node specified a URI in the 'control_path' parameter defined in Section 4.1.2.1 of {{I-D.ietf-ace-key-groupcomm}}, the Group Manager MUST inform the leaving node of its eviction, by sending a DELETE request targeting the URI specified in the 'control_path' parameter (OPT9).

If the leaving node has not exclusively the role of monitor, the Group Manager performs the following actions.

* The Group Manager frees the OSCORE Sender ID value of the leaving node, which becomes available for possible upcoming joining nodes.

* The Group Manager cancels the association between, on one hand, the public key of the leaving node and, on the other hand, the Group Identifier (Gid) associated to the OSCORE group together with the freed OSCORE Sender ID value. The Group Manager deletes the public key of the leaving node, if that public key has no remaining association with any pair (Gid, Sender ID).

If the application requires forward security, the Group Manager MUST generate updated security parameters and group keying material, and provide it to the remaining group members (see {{sec-group-rekeying-process}}). As a consequence, the leaving node is not able to acquire the new security parameters and group keying material distributed after its leaving.

Same considerations in Section 5 of {{I-D.ietf-ace-key-groupcomm}} apply here as well, considering the Group Manager acting as KDC.

# Group Rekeying Process {#sec-group-rekeying-process}

In order to rekey the OSCORE group, the Group Manager distributes a new Group Identifier (Gid), i.e. a new OSCORE ID Context; a new OSCORE Master Secret; and, optionally, a new OSCORE Master Salt for that group. When doing so, the Group Manager MUST increment the version number of the group keying material, before starting its distribution.

Furthermore, the Group Manager MUST preserve the same unchanged Sender IDs for all group members. This avoids affecting the retrieval of public keys from the Group Manager as well as the verification of message countersignatures.

The Group Manager MUST support at least the following group rekeying scheme. Future application profiles may define alternative message formats and distribution schemes.

As group rekeying message, the Group Manager uses the same format of the Joining Response message in {{ssec-join-resp}}. In particular:

* Only the parameters 'gkty', 'key', 'num', 'ace-groupcomm-profile' and 'exp' are present.

* The 'ms' parameter of the 'key' parameter specifies the new OSCORE Master Secret value.

* The 'contextId' parameter of the 'key' parameter specifies the new Group ID.

The Group Manager separately sends a group rekeying message to each group member to be rekeyed.

Each rekeying message MUST be secured with the pairwise secure communication channel between the Group Manager and the group member used during the joining process. In particular, each rekeying message can target the 'control_path' URI path defined in Section 4.1.2.1 of {{I-D.ietf-ace-key-groupcomm}} (OPT9), if provided by the intended recipient upon joining the group (see {{ssec-join-req-sending}}).

It is RECOMMENDED that the Group Manager gets confirmation of successful distribution from the group members, and admits a maximum number of individual retransmissions to non-confirming group members.

This approach requires group members to act (also) as servers, in order to correctly handle unsolicited group rekeying messages from the Group Manager. In particular, if a group member and the Group Manager use OSCORE {{RFC8613}} to secure their pairwise communications, the group member MUST create a Replay Window in its own Recipient Context upon establishing the OSCORE Security Context with the Group Manager, e.g. by means of the OSCORE profile of ACE {{I-D.ietf-ace-oscore-profile}}.

Group members and the Group Manager SHOULD additionally support alternative rekeying approaches that do not require group members to act (also) as servers. A number of such approaches are defined in Section 4.3 of {{I-D.ietf-ace-key-groupcomm}}. In particular, a group member may subscribe for updates to the group-membership resource of the group, at the endpoint /ace-group/GROUPNAME/nodes/NODENAME of the Group Manager. This can rely on CoAP Observe {{RFC7641}} or on a full-fledged Pub-Sub model {{I-D.ietf-core-coap-pubsub}} with the Group Manager acting as Broker.

In case the rekeying terminates and some group members have not received the new keying material, they will not be able to correctly process following secured messages exchanged in the group. These group members will eventually contact the Group Manager, in order to retrieve the current keying material and its version.

Some of these group members may be in multiple groups, each associated to a different Group Manager. When failing to correctly process messages secured with the new keying material, these group members may not have sufficient information to determine which exact Group Manager they should contact, in order to retrieve the current keying material they are missing.

If the Gid is formatted as described in Appendix C of {{I-D.ietf-core-oscore-groupcomm}}, the Group Prefix can be used as a hint to determine the right Group Manager, as long as no collisions among Group Prefixes are experienced. Otherwise, a group member needs to contact the Group Manager of each group, e.g. by first requesting only the version of the current group keying material (see {{sec-version}}) and then possibly requesting the current keying material (see {{ssec-updated-key-only}}).

Furthermore, some of these group members can be in multiple groups, all of which associated to the same Group Manager. In this case, these group members may also not have sufficient information to determine which exact group they should refer to, when contacting the right Group Manager. Hence, they need to contact a Group Manager multiple times, i.e. separately for each group they belong to and associated to that Group Manager.

# Default Values for Group Configuration Parameters

This section defines the default values that the Group Manager assumes for the configuration parameters of an OSCORE group, unless differently specified when creating and configuring the group. This can be achieved as specified in {{I-D.ietf-ace-oscore-gm-admin}}.

The Group Manager SHOULD use the same default values defined in Section 3.2 of {{RFC8613}} for both the HKDF algorithm and the AEAD algorithm used in the group.

The Group Manager SHOULD use the following default values for the algorithm, algorithm parameters and key parameters used to countersign messages in the group, consistently with the "COSE Algorithms" Registry {{COSE.Algorithms}}, the "COSE Key Types" Registry {{COSE.Key.Types}} and the "COSE Elliptic Curves" Registry {{COSE.Elliptic.Curves}}.

* For the algorithm 'cs_alg' used to countersign messages in the group, the signature algorithm EdDSA {{RFC8032}}.

* For the parameters 'cs_params' of the counter signature algorithm:

    - The array \[\[OKP\], \[OKP, Ed25519\]\], indicating the elliptic curve Ed25519 {{RFC8032}}, in case EdDSA is assumed or specified for 'cs_alg'.

    - The array \[\[EC2\], \[EC2, P-256\]\], indicating the elliptic curve P-256, in case ES256 {{RFC6979}} is specified for 'cs_alg'.

    - The array \[\[EC2\], \[EC2, P-384\]\], indicating the elliptic curve P-384, in case ES384 {{RFC6979}} is specified for 'cs_alg'.

    - The array \[\[EC2\], \[EC2, P-521\]\], indicating the elliptic curve P-521, in case ES512 {{RFC6979}} is specified for 'cs_alg'.

    - The array \[\[\], \[RSA\]\], in case PS256, PS384 or PS512 {{RFC8017}} is specified for 'cs_alg'.

* For the parameters 'cs_key_params' of the key used with the counter signature algorithm:

    - The array \[OKP, Ed25519\] as pair (key type, elliptic curve), in case EdDSA is assumed or specified for 'cs_alg' and Ed25519 is assumed or specified within the second array of 'cs_params'.

    - The array \[OKP, Ed448\] as pair (key type, elliptic curve), in case EdDSA is assumed or specified for 'cs_alg' and the elliptic curve Ed448 {{RFC8032}} is specified within the second array of 'cs_params'.

    - The array \[EC2, P-256\] as pair (key type, elliptic curve), in case ES256 {{RFC6979}} is specified for 'cs_alg' and the elliptic curve P-256 is assumed or specified within the second array of 'cs_params'.

    - The array \[EC2, P-384\] as pair (key type, elliptic curve), in case ES384 {{RFC6979}} is specified for 'cs_alg' and the elliptic curve P-384 is specified within the second array of 'cs_params'.

    - The array \[EC2, P-521\] as pair (key type, elliptic curve), in case ES512 {{RFC6979}} is specified for 'cs_alg' and the elliptic curve P-521 is specified within the second array of 'cs_params'.

    - The array \[RSA\] indicating RSA as key type, in case PS256, PS384 or PS512 {{RFC8017}} is specified for 'cs_alg'.

* For the 'cs_key_enc' encoding of the public keys of the group members, COSE_Key from the "CWT Confirmation Methods" Registry {{CWT.Confirmation.Methods}}.

# Security Considerations {#sec-security-considerations}

Security considerations for this profile are inherited from {{I-D.ietf-ace-key-groupcomm}}, the ACE framework for Authentication and Authorization {{I-D.ietf-ace-oauth-authz}}, and the specific transport profile of ACE signalled by the AS, such as {{I-D.ietf-ace-dtls-authorize}} and {{I-D.ietf-ace-oscore-profile}}.

The following security considerations also apply for this profile.

## Management of OSCORE Groups {#ssec-security-considerations-management}

This profile leverages the following management aspects related to OSCORE groups and discussed in the sections of {{I-D.ietf-core-oscore-groupcomm}} referred below.

* Management of group keying material (see Section 3.1 of {{I-D.ietf-core-oscore-groupcomm}}). The Group Manager is responsible for the renewal and re-distribution of the keying material in the groups of its competence (rekeying). According to the specific application requirements, this can include rekeying the group upon changes in its membership. In particular, renewing the group keying material is required upon a new node's joining or a current node's leaving, in case backward security and forward security have to be preserved, respectively.

* Provisioning and retrieval of public keys (see Section 2 of {{I-D.ietf-core-oscore-groupcomm}}). The Group Manager acts as key repository of public keys of group members, and provides them upon request.

* Synchronization of sequence numbers (see Section 6.1 of {{I-D.ietf-core-oscore-groupcomm}}). This concerns how a responder node that has just joined an OSCORE group can synchronize with the sequence number of requesters in the same group.

Before sending the Joining Response, the Group Manager MUST verify that the joining node actually owns the associated private key. To this end, the Group Manager can rely on the proof-of-possession challenge-response defined in {{sec-joining-node-to-GM}}. Alternatively, the joining node can use its own public key as asymmetric proof-of-possession key to establish a secure channel with the Group Manager, e.g. as in Section 3.2 of {{I-D.ietf-ace-dtls-authorize}}. However, this requires such proof-of-possession key to be compatible with the encoding as well as with the countersignature algorithm and possible associated parameters used in the OSCORE group.

A node may have joined multiple OSCORE groups under different non-synchronized Group Managers. Therefore, it can happen that those OSCORE groups have the same Group Identifier (Gid). It follows that, upon receiving a Group OSCORE message addressed to one of those groups, the node would have multiple Security Contexts matching with the Gid in the incoming message. It is up to the application to decide how to handle such collisions of Group Identifiers, e.g. by trying to process the incoming message using one Security Context at the time until the right one is found.

## Size of Nonces for Signature Challenge {#ssec-security-considerations-challenges}

With reference to the Joining Request message in {{ssec-join-req-sending}}, the proof-of-possession signature included in 'client\_cred\_verify' is computed over the challenge N\_C \| N_S, where \| denotes concatenation.

For the N\_C challenge share, it is RECOMMENDED to use a 8-byte long random nonce. Furthermore, N\_C is always conveyed in the 'cnonce' parameter of the Joining Request, which is always sent over the secure communication channel  between the joining node and the Group Manager.

As defined in {{sssec-challenge-value}}, the way the N\_S value is computed depends on the particular way the joining node provides the Group Manager with the Access Token, as well as on following interactions between the two.

* If the Access Token is not explicitly posted to the /authz-info endpoint of the Group Manager, then N\_S is computed as a 32-byte long challenge share (see points 2 of {{sssec-challenge-value}}).

* If the Access Token has been explicitly posted to the /authz-info endpoint of the Group Manager, N\_S takes the most recent value specified to the client by the Group Manager in the 'kdcchallenge' parameter (see point 1 of {{sssec-challenge-value}}). This is specified either in the 2.01 response to the Token Post (see {{ssec-token-post}}), or in a 4.00 response to a following Joining Request (see {{ssec-join-req-processing}}). In either case, it is RECOMMENDED to use a 8-byte long random challenge as value for N\_S.

If we consider both N\_C and N\_S to take 8-byte long values, the following considerations hold.

* Let us consider both N\_C and N\_S as taking random values, and the Group Manager to never change the value of the N\_S provided to a Client during the lifetime of an Access Token. Then, as per the birthday paradox, the average collision for N\_S will happen after 2^32 new posted Access Tokens, while the average collision for N\_C will happen after 2^32 new Joining Requests. This amounts to considerably more token provisionings than the expected new joinings of OSCORE groups under a same Group Manager, as well as to considerably more requests to join OSCORE groups from a same Client using a same Access Token under a same Group Manager.

* Section 7 of {{I-D.ietf-ace-oscore-profile}} as well Appendix B.2 of {{RFC8613}} recommend the use of 8-byte random values as well. Unlike in those cases, the values of N\_C and N\_S considered in this specification are not used for as sensitive operations as the derivation of a Security Context, with possible implications in the security of AEAD ciphers.

## Reusage of Nonces for Signature Challenge {#ssec-security-considerations-reusage-nonces}

As long as the Group Manager preserves the same N\_S value currently associated to an Access Token, i.e. the latest value provided to a Client in a 'kdcchallenge' parameter, the Client is able to successfully reuse the same signature challenge for multiple Joining Requests to that Group Manager.

In particular, the client can reuse the same N\_C value for every Joining Request to the Group Manager, and combine it with the same unchanged N\_S value. This results in reusing the same signature challenge for producing the signature to include in the 'client_cred_verify' parameter of the Joining Requests.

Unless the Group Manager maintains a list of N\_C values already used by that Client since the latest update to the N\_S value associated to the Access Token, the Group Manager can be forced to falsely believe that the Client possesses its own private key at that point in time, upon verifying the signature in the 'client_cred_verify' parameter.

# IANA Considerations {#sec-iana}

Note to RFC Editor: Please replace all occurrences of "\[\[This specification\]\]" with the RFC number of this specification and delete this paragraph.

This document has the following actions for IANA.

## ACE Groupcomm Profile Registry {#ssec-iana-groupcomm-profile-registry}

IANA is asked to register the following entry to the "ACE Groupcomm Profile" Registry defined in Section 8.7 of {{I-D.ietf-ace-key-groupcomm}}.

*  Name: coap_group_oscore_app
*  Description: Application profile to provision keying material for participating in group communication protected with Group OSCORE as per {{I-D.ietf-core-oscore-groupcomm}}.
*  CBOR Value: TBD1
*  Reference: \[\[This specification\]\] ({{ssec-join-resp}})

## ACE Groupcomm Key Registry {#ssec-iana-groupcomm-key-registry}

IANA is asked to register the following entry to the "ACE Groupcomm Key" Registry defined in Section 8.6 of {{I-D.ietf-ace-key-groupcomm}}.

*  Name: Group_OSCORE_Security_Context object
*  Key Type Value: TBD2
*  Profile: "coap_group_oscore_app", defined in {{ssec-iana-groupcomm-profile-registry}} of this specification.
*  Description: A Group_OSCORE_Security_Context object encoded as described in {{ssec-join-resp}} of this specification.
*  Reference: \[\[This specification\]\] ({{ssec-join-resp}})

## OSCORE Security Context Parameters Registry {#ssec-iana-security-context-parameter-registry}

IANA is asked to register the following entries in the "OSCORE Security Context Parameters" Registry defined in Section 9.4 of {{I-D.ietf-ace-oscore-profile}}.

*  Name: cs_alg
*  CBOR Label: TBD3
*  CBOR Type: tstr / int
*  Registry: COSE Algorithm Values (ECDSA, EdDSA)
*  Description: OSCORE Counter Signature Algorithm Value
*  Reference: \[\[This specification\]\] ({{ssec-join-resp}})

~~~~~~~~~~~

~~~~~~~~~~~

*  Name: cs_params
*  CBOR Label: TBD4
*  CBOR Type: array
*  Registry: Counter Signatures Parameters
*  Description: OSCORE Counter Signature Algorithm Additional Parameters
*  Reference: \[\[This specification\]\] ({{ssec-join-resp}})

~~~~~~~~~~~

~~~~~~~~~~~

*  Name: cs_key_params
*  CBOR Label: TBD5
*  CBOR Type: array
*  Registry: Counter Signatures Key Parameters
*  Description: OSCORE Counter Signature Key Additional Parameters
*  Reference: \[\[This specification\]\] ({{ssec-join-resp}})

~~~~~~~~~~~

~~~~~~~~~~~

*  Name: cs_key_enc
*  CBOR Label: TBD6
*  CBOR Type: integer
*  Registry: ACE Public Key Encoding
*  Description: Encoding of Public Keys to be used with the OSCORE Counter Signature Algorithm
*  Reference: \[\[This specification\]\] ({{ssec-join-resp}})

## Sequence Number Synchronization Method Registry {#ssec-iana-sn-synch-method-registry}

IANA is asked to register the following entries in the "Sequence Number Synchronization Method" Registry defined in Section 8.9 of {{I-D.ietf-ace-key-groupcomm}}.

*  Name: Best effort
*  Value: 1
*  Description: No action is taken.
*  Reference: {{I-D.ietf-core-oscore-groupcomm}} (Appendix E.1)

~~~~~~~~~~~

~~~~~~~~~~~

*  Name: Baseline
*  Value: 2
*  Description: The first received request sets the baseline reference point, and is discarded with no delivery to the application.
*  Reference: {{I-D.ietf-core-oscore-groupcomm}} (Appendix E.2)

~~~~~~~~~~~

~~~~~~~~~~~

*  Name: Echo challenge-response
*  Value: 3
*  Description: Challenge response using the Echo Option for CoAP from {{I-D.ietf-core-echo-request-tag}}.
*  Reference: {{I-D.ietf-core-oscore-groupcomm}} (Appendix E.3)

## ACE Groupcomm Parameters Registry {#ssec-iana-ace-groupcomm-parameters-registry}

IANA is asked to register the following entry to the "ACE Groupcomm Parameters" Registry defined in Section 8.5 of {{I-D.ietf-ace-key-groupcomm}}.

* Name: clientId
* CBOR Key: TBD7
* CBOR Type: Byte string
* Reference: \[\[This specification\]\] ({{sec-new-key}})

## ACE Groupcomm Policy Registry {#ssec-iana-ace-groupcomm-policy-registry}

IANA is asked to register the following entry to the "ACE Groupcomm Policy" Registry defined in Section 8.8 of {{I-D.ietf-ace-key-groupcomm}}.

* Name: Group OSCORE Pairwise Mode Support
* CBOR Key: TBD8
* CBOR Type: Simple value
* Description: True if the OSCORE group supports the pairwise mode of Group OSCORE {{I-D.ietf-core-oscore-groupcomm}}, False otherwise.
* Reference: \[\[This specification\]\] ({{ssec-pairwise-mode-policy}})

## TLS Exporter Label Registry {#ssec-iana-tls-esporter-label-registry}

IANA is asked to register the following entry to the "TLS Exporter Label" Registry defined in Section 6 of {{RFC5705}} and updated in Section 12 of {{RFC8447}}.

* Value: EXPORTER-ACE-Sign-Challenge-coap-group-oscore-app
* DTLS-OK: Y
* Recommended: N
* Reference: \[\[This specification\]\] ({{sssec-challenge-value}})

## AIF Registry {#ssec-iana-AIF-registry}

IANA is asked to register the following entry to the "Toid" sub-registry of the "AIF" Registry defined in Section 5.2 of {{I-D.bormann-core-ace-aif}}.

* Name: oscore-group-name
* Description/Specification: group name of the OSCORE group, as specified in \[\[This specification\]\].

IANA is asked to register the following entry to the "Tperm" sub-Registry of the "AIF" Registry defined in Section 5.2 of {{I-D.bormann-core-ace-aif}}.

* Name: oscore-group-roles
* Description/Specification: role(s) of the member of the OSCORE group, as specified in \[\[This specification\]\].

## Media Type Registrations {#ssec-iana-media-types}

This specification registers the 'application/aif-groupcomm-oscore+cbor' media type for the AIF specific data model AIF-OSCORE-GROUPCOMM defined in {{sec-format-scope}} of \[\[This specification\]\]. This registration follows the procedures specified in {{RFC6838}}.

These media type has parameters for specifying the object identifier ("Toid") and set of permissions ("Tperm") defined for the AIF-generic model in {{I-D.bormann-core-ace-aif}}; default values are the values "oscore-group-name" for "Toid" and "oscore-group-roles" for "Tperm".

Type name: application

Subtype name: aif-groupcomm-oscore+cbor

Required parameters: "Toid", "Tperm"

Optional parameters: none

Encoding considerations: Must be encoded as a CBOR array, each element of which is an array \[Toid, Tperm\] as defined in {{sec-format-scope}} of \[\[This specification\]\].

Security considerations: See {{sec-security-considerations}} of \[\[This specification\]\].

Interoperability considerations: n/a

Published specification: \[\[This specification\]\]

Applications that use this media type: The type is used by applications that want to express authorization information about joining OSCORE groups, as specified in \[\[This specification\]\].

Additional information: n/a

Person & email address to contact for further information: <iesg@ietf.org>

Intended usage: COMMON

Restrictions on usage: None

Author: Marco Tiloca <marco.tiloca@ri.se>

Change controller: IESG

## CoAP Content-Format Registry {#ssec-iana-coap-content-format-registry}

IANA is asked to register the following entry to the "CoAP Content-Formats" registry, within the "CoRE Parameters" registry:

Media Type: application/aif-groupcomm-oscore+cbor;Toid="oscore-group-name",Tperm"oscore-group-roles"

Encoding: -

ID: TBD9

Reference: \[\[This specification\]\]

## Group OSCORE Roles Registry {#ssec-iana-group-oscore-roles-registry}

This specification establishes the IANA "Group OSCORE Roles" Registry. The Registry has been created to use the "Expert Review Required" registration procedure {{RFC8126}}. Expert review guidelines are provided in {{ssec-iana-expert-review}}.

This registry includes the possible roles that nodes can take in an OSCORE group, each in combination with a numeric identifier. These numeric identifiers are used to express authorization information about joining OSCORE groups, as specified in {{sec-format-scope}} of \[\[This specification\]\].

The columns of this registry are:

* Name: A value that can be used in documents for easier comprehension, to identify a possible role that nodes can take in an OSCORE group.

* Value: The numeric identifier for this role. Integer values less than -65536 are marked as "Private Use", all other values use the registration policy "Expert Review" {{RFC8126}}.

* Description: This field contains a brief description of the role.

* Reference: This contains a pointer to the public specification for the role.

This registry will be initially populated by the values in {{fig-role-values}}.

The Reference column for all of these entries will be \[\[This specification\]\].

## CoRE Resource Type Registry # {#iana-rt}

IANA is asked to register a new Resource Type (rt=) Link Target Attribute in the  "Resource Type (rt=) Link Target Attribute Values" subregistry under the "Constrained Restful Environments (CoRE) Parameters" {{CORE.Parameters}} registry.

* Value: "core.osc.gm"

* Description: Group-membership resource of an OSCORE Group Manager.

* Reference: \[\[This specification\]\]

## Expert Review Instructions {#ssec-iana-expert-review}

The IANA Registry established in this document is defined as "Expert Review".  This section gives some general guidelines for what the experts should be looking for, but they are being designated as experts for a reason so they should be given substantial latitude.

Expert reviewers should take into consideration the following points:

* Clarity and correctness of registrations. Experts are expected to check the clarity of purpose and use of the requested entries. Experts should inspect the entry for the considered role, to verify the correctness of its description against the role as intended in the specification that defined it. Expert should consider requesting an opinion on the correctness of registered parameters from the Authentication and Authorization for Constrained Environments (ACE) Working Group and the Constrained RESTful Environments (CoRE) Working Group.

     Entries that do not meet these objective of clarity and completeness should not be registered.

* Duplicated registration and point squatting should be discouraged. Reviewers are encouraged to get sufficient information for registration requests to ensure that the usage is not going to duplicate one that is already registered and that the point is likely to be used in deployments.

* Experts should take into account the expected usage of roles when approving point assignment. Given a 'Value' V as code point, the length of the encoding of (2^(V+1) - 1) should be weighed against the usage of the entry, considering the resources and capabilities of devices it will be used on. Additionally, given a 'Value' V as code point, the length of the encoding of (2^(V+1) - 1) should be weighed against how many code points resulting in that encoding length are left, and the resources and capabilities of devices it will be used on.

* Specifications are recommended. When specifications are not provided, the description provided needs to have sufficient information to verify the points above.

--- back

# Profile Requirements # {#profile-req}

This appendix lists the specifications on this application profile of ACE, based on the requirements defined in Appendix A of {{I-D.ietf-ace-key-groupcomm}}.

* REQ1 - If the value of the GROUPNAME URI path and the group name in the Access Token scope (gname in Section 3.1 of {{I-D.ietf-ace-key-groupcomm}}) do not match, specify the mechanism to map the GROUPNAME value in the URI to the group name: not applicable, since a match is required.

* REQ2 - Specify the encoding and value of roles, for scope entries of 'scope': see {{sec-format-scope}} and {{ssec-auth-req}}.

* REQ3 - if used, specify the acceptable values for 'sign\_alg': values from the "Value" column of the "COSE Algorithms" Registry {{COSE.Algorithms}}.

* REQ4 - If used, specify the acceptable values for 'sign\_parameters': values from the COSE capabilities in the "COSE Algorithms" Registry {{COSE.Algorithms}} and from the COSE capabilities in the "COSE Key Types" Registry {{COSE.Key.Types}}.

* REQ5 - If used, specify the acceptable values for 'sign\_key\_parameters': values from the COSE capabilities in the "COSE Key Types" Registry {{COSE.Key.Types}}.

* REQ6 - If used, specify the acceptable values for 'pub\_key\_enc': 1 ("COSE\_Key") from the 'Confirmation Key' column of the "CWT Confirmation Method" Registry {{CWT.Confirmation.Methods}}. Future specifications may define additional values for this parameter.

* REQ7a - Register a Resource Type for the root url-path, which is used to discover the correct url to access at the KDC: the Resource Type (rt=) Link Target Attribute value "core.osc.gm" is registered in {{iana-rt}}.

* REQ7 - Format of the 'key' value: see {{ssec-join-resp}}.

* REQ8 - Acceptable values of 'gkty': Group_OSCORE_Security_Context object (see {{ssec-join-resp}}).

* REQ9: Specify the format of the identifiers of group members: see {{ssec-join-resp}} and {{sec-pub-keys}}.

* REQ10 - Specify the communication protocol that the members of the group must use: CoAP, possibly over IP multicast.

* REQ11 - Specify the security protocols that the group members must use to protect their communication: Group OSCORE.

* REQ12 - Specify and register the application profile identifier: coap_group_oscore_app (see {{ssec-iana-groupcomm-profile-registry}}).

* REQ13 - Specify policies at the KDC to handle member ids that are not included in 'get_pub_keys': see {{sec-pub-keys}}.

* REQ14 - If used, specify the content format and default value of 'group\_policies' and its entries: see {{ssec-join-resp}}; the three values defined and registered as content of the entry "Sequence Number Synchronization Method" (see {{ssec-iana-sn-synch-method-registry}}); the defined and registered encoding of the entry "Group OSCORE Pairwise Mode Support" (see {{ssec-iana-ace-groupcomm-policy-registry}}).

* REQ15 - Specify the format of newly-generated individual keying material for group members, or of the information to derive it, and corresponding CBOR label: see {{sec-new-key}}.

* REQ16 - Specify how the communication is secured between the Client and KDC: by means of any transport profile of ACE {{I-D.ietf-ace-oauth-authz}} between Client and Group Manager that complies with the requirements in Appendix C of {{I-D.ietf-ace-oauth-authz}}.

* REQ17 - Specify how the nonce N\_S is generated, if the token is not being posted (e.g. if it is used directly to validate TLS instead): see {{sssec-challenge-value}}.

* REQ18 - Specify if 'mgt_key_material' used, and if yes specify its format and content: not used in this version of the profile.

* REQ19 - Define the initial value of the 'num' parameter: The initial value MUST be set to 0 when creating the OSCORE group, e.g. as in {{I-D.ietf-ace-oscore-gm-admin}}.

* OPT1 (Optional) - Specify the encoding of public keys, of 'client\_cred', and of 'pub\_keys' if COSE_Keys are not used: no.

* OPT2 (Optional) - Specify the negotiation of parameter values for signature algorithm and signature keys, if 'sign_info' is not used: possible early discovery by using the approach based on the CoRE Resource Directory described in {{I-D.tiloca-core-oscore-discovery}}.

* OPT3 (Optional) - Specify the encoding of 'pub_keys_repos' if the default is not used: no.

* OPT4 (Optional) - Specify policies that instruct clients to retain unsuccessfully decrypted messages and for how long, so that they can be decrypted after getting updated keying material: no.

* OPT5 (Optional) - Specify the behavior of the handler in case of failure to retrieve a public key for the specific node: send a 4.00 Bad Request response to a Joining Request (see {{ssec-join-req-processing}}).

* OPT6 (Optional) - Specify possible or required payload formats for specific error cases: send a 4.00 Bad Request response to a Joining Request (see {{ssec-join-req-processing}}).

* OPT7 (Optional) - Specify CBOR values to use for abbreviating identifiers of roles in the group or topic (see {{ssec-auth-req}}).

* OPT8 (Optional) - Specify for the KDC to perform group rekeying (together or instead of renewing individual keying material) when receiving a Key Renewal Request: the Group Manager SHOULD NOT perform a group rekeying, unless already scheduled to occur shortly (see {{sec-new-key}}).

* OPT9 (Optional) - Specify the functionalities implemented at the 'control_path' resource hosted at the Client, including message exchange encoding and other details (see Section 4.1.2.1 of {{I-D.ietf-ace-key-groupcomm}}): see {{sec-leaving}} for the eviction of a group member; see {{sec-group-rekeying-process}} for the group rekeying process.

* OPT10 (Optional) - Specify how the identifier of the sender's public key is included in the group request: no.

# Document Updates # {#sec-document-updates}

RFC EDITOR: PLEASE REMOVE THIS SECTION.

## Version -08 to -09 ## {#sec-08-09}

* The url-path "ace-group" is used.

* The signed value for 'client_cred_verify' includes also the scope.

* Clarified non intended meanings of 'clientId'.

* Updates on group rekeying contextual with request of new Sender ID.

* Registration of the resource type rt="core.osc.gm".

## Version -07 to -08 ## {#sec-07-08}

* AIF specific data model to express scope entries.

* A set of roles is checked as valid when processing the Joining Request.

* Updated format of 'get_pub_keys' in the Joining Request.

* Payload format and default values of group policies in the Joining Response.

* Updated payload format of the FETCH request to retrieve public keys.

* Default values for group configuration parameters.

* IANA registrations to support the AIF specific data model.

## Version -06 to -07 ## {#sec-06-07}

* Alignments with draft-ietf-core-oscore-groupcomm.

* New format of 'sign_info', using the COSE capabilities.

* New format of Joining Response parameters, using the COSE capabilities.

* Considerations on group rekeying.

* Editorial revision.

## Version -05 to -06 ## {#sec-05-06}

* Added role of external signature verifier.

* Parameter 'rsnonce' renamed to 'kdcchallenge'.

* Parameter 'kdcchallenge' may be omitted in some cases.

* Clarified difference between group name and OSCORE Gid.

* Removed the role combination \["requester", "monitor"\].

* Admit implicit scope and audience in the Authorization Request.

* New format for the 'sign_info' parameter.

* Scope not mandatory to include in the Joining Request.

* Group policy about supporting Group OSCORE in pairwise mode.

* Possible individual rekeying of a single requesting node combined with a group rekeying.

* Security considerations on reusage of signature challenges.

* Addressing optional requirement OPT9 from draft-ietf-ace-key-groupcomm

* Editorial improvements.

## Version -04 to -05 ## {#sec-04-05}

* Nonce N\_S also in error responses to the Joining Requests.

* Supporting single Access Token for multiple groups/topics.

* Supporting legal requesters/responders using the 'peer_roles' parameter.

* Registered and used dedicated label for TLS Exporter.

* Added method for uploading a new public key to the Group Manager.

* Added resource and method for retrieving the current group status.

* Fixed inconsistency in retrieving group keying material only.

* Clarified retrieval of keying material for monitor-only members.

* Clarification on incrementing version number when rekeying the group.

* Clarification on what is re-distributed with the group rekeying.

* Security considerations on the size of the nonces used for the signature challenge.

* Added CBOR values to abbreviate role identifiers in the group.

## Version -03 to -04 ## {#sec-03-04}

* New abstract.

* Moved general content to draft-ietf-ace-key-groupcomm

* Terminology: node name; node resource.

* Creation and pointing at node resource.

* Updated Group Manager API (REST methods and offered services).

* Size of challenges 'cnonce' and 'rsnonce'.

* Value of 'rsnonce' for reused or non-traditionally-posted tokens.

* Removed reference to RFC 7390.

* New requirements from draft-ietf-ace-key-groupcomm

* Editorial improvements.

## Version -02 to -03 ## {#sec-02-03}

* New sections, aligned with the interface of ace-key-groupcomm .

* Exchange of information on the countersignature algorithm and related parameters, during the Token POST (Section 4.1).

* Nonce 'rsnonce' from the Group Manager to the Client (Section 4.1).

* Client PoP signature in the Key Distribution Request upon joining (Section 4.2).

* Local actions on the Group Manager, upon a new node's joining (Section 4.2).

* Local actions on the Group Manager, upon a node's leaving (Section 12).

* IANA registration in ACE Groupcomm Parameters Registry.

* More fulfilled profile requirements (Appendix A).

## Version -01 to -02 ## {#sec-01-02}

* Editorial fixes.

* Changed: "listener" to "responder"; "pure listener" to "monitor".

* Changed profile name to "coap_group_oscore_app", to reflect it is an application profile.

* Added the 'type' parameter for all requests to a Join Resource.

* Added parameters to indicate the encoding of public keys.

* Challenge-response for proof-of-possession of signature keys (Section 4).

* Renamed 'key_info' parameter to 'sign_info'; updated its format; extended to include also parameters of the countersignature key (Section 4.1).

* Code 4.00 (Bad request), in responses to joining nodes providing an invalid public key (Section 4.3).

* Clarifications on provisioning and checking of public keys (Sections 4 and 6).

* Extended discussion on group rekeying and possible different approaches (Section 7).

* Extended security considerations: proof-of-possession of signature keys; collision of OSCORE Group Identifiers (Section 8).

* Registered three entries in the IANA Registry "Sequence Number Synchronization Method Registry" (Section 9).

* Registered one public key encoding in the "ACE Public Key Encoding" IANA Registry (Section 9).

## Version -00 to -01 ## {#sec-00-01}

* Changed name of 'req_aud' to 'audience' in the Authorization Request (Section 3.1).

* Added negotiation of countersignature algorithm/parameters between Client and Group Manager (Section 4).

* Updated format of the Key Distribution Response as a whole (Section 4.3).

* Added parameter 'cs_params' in the 'key' parameter of the Key Distribution Response (Section 4.3).

* New IANA registrations in the "ACE Authorization Server Request Creation Hints" Registry, "ACE Groupcomm Key" Registry, "OSCORE Security Context Parameters" Registry and "ACE Groupcomm Profile" Registry (Section 9).

# Acknowledgments {#sec-acknowledgments}
{: numbered="no"}

The authors sincerely thank Santiago Arag&oacute;n, Stefan Beck, Carsten Bormann, Martin Gunnarsson, Rikard Hoeglund, Daniel Migault, Jim Schaad, Ludwig Seitz, Goeran Selander and Peter van der Stok for their comments and feedback.

The work on this document has been partly supported by VINNOVA and the Celtic-Next project CRITISEC; and by the EIT-Digital High Impact Initiative ACTIVE.

--- fluff
