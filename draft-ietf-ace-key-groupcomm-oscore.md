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
  RFC3986:
  RFC5869:
  RFC7252:
  RFC8152:
  RFC8174:
  RFC8446:
  RFC8613:
  I-D.ietf-core-oscore-groupcomm:
  I-D.ietf-ace-key-groupcomm:
  I-D.ietf-ace-oauth-authz:
  I-D.ietf-ace-oscore-profile:
  I-D.ietf-ace-cwt-proof-of-possession:

informative:
  I-D.dijk-core-groupcomm-bis:
  I-D.ietf-ace-dtls-authorize:
  I-D.ietf-core-coap-pubsub:
  I-D.tiloca-core-oscore-discovery:
  I-D.ietf-core-echo-request-tag:
  I-D.ietf-ace-mqtt-tls-profile:
  I-D.ietf-ace-dtls-authorize:
  RFC6347:
  RFC6749:
  RFC7641:

--- abstract

This specification defines an application profile of the ACE framework for Authentication and Authorization, to request and provision keying material in group communication scenarios that are based on CoAP and secured with Group Object Security for Constrained RESTful Environments (OSCORE). This application profile delegates the authentication and authorization of Clients that join an OSCORE group through a Resource Server acting as Group Manager for that group. This application profile leverages protocol-specific transport profiles of ACE to achieve communication security, server authentication and proof-of-possession for a key owned by the Client and bound to an OAuth 2.0 access token.

--- middle

# Introduction {#sec-introduction}

Object Security for Constrained RESTful Environments (OSCORE) {{RFC8613}} is a method for application-layer protection of the Constrained Application Protocol (CoAP) {{RFC7252}}, using CBOR Object Signing and Encryption (COSE) {{RFC8152}} and enabling end-to-end security of CoAP payload and options.

As described in {{I-D.ietf-core-oscore-groupcomm}}, Group OSCORE is used to protect CoAP group communication over IP multicast {{I-D.dijk-core-groupcomm-bis}}. This relies on a Group Manager, which is responsible for managing an OSCORE group, where members exchange CoAP messages secured with Group OSCORE. The Group Manager can be responsible for multiple groups, coordinates the joining process of new group members, and is entrusted with the distribution and renewal of group keying material.

<!-- 13-01-2020 FP: Covered in ace-key-groupcomm

This specification builds on the ACE framework for Authentication and Authorization {{I-D.ietf-ace-oauth-authz}} and defines an application profile of ACE to:

* Authorize a node to join an OSCORE group, and provide it with the group keying material to communicate with other group members.

* Provide updated keying material to group members upon request.

* Renew the group keying material and distribute it to the OSCORE group (rekeying) upon changes in the group membership.

A client node joins an OSCORE group through a resource server acting as Group Manager for that group. The joining process relies on an Access Token, which is bound to a proof-of-possession key and authorizes the client to access a specific group-membership resource at the Group Manager.

-->


This specification is an application profile of {{I-D.ietf-ace-key-groupcomm}}, which itself builds on the ACE framework for Authentication and Authorization {{I-D.ietf-ace-oauth-authz}}. Message exchanges among the participants as well as message formats and processing follow what specified in {{I-D.ietf-ace-key-groupcomm}} for provisioning and renewing keying material in group communication scenarios, where Group OSCORE is used to protect CoAP group communication over IP multicast.


<!-- 13-01-2020 FP: Covered in ace-key-groupcomm

In order to achieve communication security, proof-of-possession and server authentication, the client and the Group Manager leverage protocol-specific transport profiles of ACE. These include also possible forthcoming transport profiles that comply with the requirements in Appendix C of {{I-D.ietf-ace-oauth-authz}}.

-->

## Terminology {#ssec-terminology}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}}{{RFC8174}} when, and only when, they appear in all capitals, as shown here.

Readers are expected to be familiar with:

* The terms and concepts described in the ACE framework for authentication and authorization {{I-D.ietf-ace-oauth-authz}}. The terminology for entities in the considered architecture is defined in OAuth 2.0 {{RFC6749}}. In particular, this includes Client (C), Resource Server (RS), and Authorization Server (AS).

* The terms and concepts related to the CoAP protocol described in {{RFC7252}}{{I-D.dijk-core-groupcomm-bis}}. Unless otherwise indicated, the term "endpoint" is used here following its OAuth definition, aimed at denoting resources such as /token and /introspect at the AS and /authz-info at the RS. This document does not use the CoAP definition of "endpoint", which is "An entity participating in the CoAP protocol".

* The terms and concept related to the message formats and processing specified in {{I-D.ietf-ace-key-groupcomm}}, for provisioning and renewing keying material in group communication scenarios.

* The terms and concepts for protection and processing of CoAP messages through OSCORE {{RFC8613}} and through Group OSCORE {{I-D.ietf-core-oscore-groupcomm}} in group communication scenarios. These include the concept of Group Manager, as the entity responsible for a set of groups where communications are secured with Group OSCORE. In this specification, the Group Manager acts as Resource Server.

Additionally, this document makes use of the following terminology.

* Group name is used as a synonym for group identifier in {{I-D.ietf-ace-key-groupcomm}}.

* GROUPNAME and NODENAME are used to indicate the variants parts of the resource endpoint, i.e. "gid" and "node" URI path in {{I-D.ietf-ace-key-groupcomm}}.

<!-- 13-01-2020 FP: Covered in ace-key-groupcomm

This document refers also to the following terminology.

* Joining node: a network node intending to join an OSCORE group, where communication is based on CoAP {{I-D.dijk-core-groupcomm-bis}} and secured with Group OSCORE {{I-D.ietf-core-oscore-groupcomm}}.

* Joining process: the process through which a joining node becomes a member of an OSCORE group. The joining process is enforced and assisted by the Group Manager responsible for that group.

* Group name: stable and invariant name of an OSCORE group. The group name MUST be unique under the same Group Manager, and MUST include only characters that are valid for a url-path segment, namely unreserved and pct-encoded characters {{RFC3986}}.

* Node name: stable and invariant name of a group member, assigned by the Group Manager upon joining. The group name MUST be individual and unique among the members of a same OSCORE group, and MUST include only characters that are valid for a url-path segment, namely unreserved and pct-encoded characters {{RFC3986}}.

* Group-membership resource: a resource hosted by the Group Manager, associated to an OSCORE group under that Group Manager. A group-membership resource is identifiable with the name of the respective OSCORE group. A joining node accesses a group-membership resource to start the joining process and become a member of that group. The url-path of a group-membership resource is fixed, and ends with the segments /group-oscore/GROUPNAME , where "GROUPNAME" is the name of the associated OSCORE group. This replaces the url-path /ace-group/gid at the KDC used in {{I-D.ietf-ace-key-groupcomm}}, with "gid" indicating the group name. The url-path /group-oscore/GROUPNAME is a default name: implementations are not required to use this name, and can define their own instead.

* Node resource: a resource hosted by the Group Manager, associated to a specific group member of an OSCORE group under that Group Manager. A node resource is identifiable with the name of the respective group member, which can access it to issue individual requests to the Group Manager. The url-path of a node resource is fixed, and ends with the segments /group-oscore/GROUPNAME/NODENAME , where "GROUPNAME" is the name of the associated OSCORE group and "NODENAME" is the name of the associated group member in that group. This replaces the url-path /ace-group/gid/node at the KDC used in {{I-D.ietf-ace-key-groupcomm}}, with "gid" indicating the group name and "node" indicating the node name. The url-path /group-oscore/GROUPNAME/NODENAME is a default name: implementations are not required to use this name, and can define their own instead.

* Group-membership endpoint: an endpoint at the Group Manager associated to a group-membership resource.

* Node endpoint: an endpoint at the Group Manager associated to a node resource.

* Requester: member of an OSCORE group that sends request messages to other members of the group.

* Responder: member of an OSCORE group that receives request messages from other members of the group. A responder may reply back, by sending a response message to the requester which has sent the request message.

* Monitor: member of an OSCORE group that is configured as responder and never replies back to requesters after receiving request messages. This corresponds to the term "silent server" used in {{I-D.ietf-core-oscore-groupcomm}}.

* Group rekeying process: the process through which the Group Manager renews the security parameters and group keying material, and (re-)distributes them to the OSCORE group members.

-->

# Protocol Overview {#sec-protocol-overview}

Group communication for CoAP over IP multicast has been enabled in {{I-D.dijk-core-groupcomm-bis}} and can be secured with Group Object Security for Constrained RESTful Environments (OSCORE) {{RFC8613}} as described in {{I-D.ietf-core-oscore-groupcomm}}. A network node joins an OSCORE group by interacting with the responsible Group Manager. Once registered in the group, the new node can securely exchange messages with other group members.

This specification describes how to use {{I-D.ietf-ace-key-groupcomm}} and {{I-D.ietf-ace-oauth-authz}} to perform a number of authentication, authorization and key distribution actions, as defined in Section 2. of {{I-D.ietf-ace-key-groupcomm}}, for an OSCORE group.

<!-- 13-01-2020 FP: Covered in ace-key-groupcomm

This specification describes how to use the ACE framework for authentication and authorization {{I-D.ietf-ace-oauth-authz}} to:

* Enable a node to join an OSCORE group through the Group Manager and receive the security parameters and keying material to communicate with the other members of the group.

* Enable members of OSCORE groups to retrieve updated group keying material and public key of other group members, from the Group Manager.

* Enable the Group Manager to renew the security parameters and group keying material, and to (re-)distribute them to the members of the OSCORE group (rekeying).

-->

With reference to {{I-D.ietf-ace-key-groupcomm}} :

* The node wishing to joing the OSCORE group is the Client.

* The Authorization Server is the AS.

* The Group Manager is the Key Distribution Center (KDC). 

<!-- 13-01-2020 FP: Covered in ace-key-groupcomm

With reference to the ACE framework and the terminology defined in OAuth 2.0 {{RFC6749}}:

* The Group Manager acts as Resource Server (RS), and hosts one group-membership resource for each OSCORE group it manages. Each group-membership resource is exported by a distinct group-membership endpoint. During the joining process, the Group Manager provides joining nodes with the parameters and keying material for taking part to secure communications in the OSCORE group. The Group Manager also maintains the group keying material and performs the group rekeying process to distribute updated keying material to the group members.

* The joining node acts as Client (C), and requests to join an OSCORE group by accessing the related group-membership endpoint at the Group Manager.

* The Authorization Server (AS) authorizes joining nodes to join OSCORE groups under their respective Group Manager. Multiple Group Managers can be associated to the same AS. The AS MAY release Access Tokens for other purposes than joining OSCORE groups under registered Group Managers. For example, the AS may also release Access Tokens for accessing resources hosted by members of OSCORE groups.

-->

All communications between the involved entities rely on the CoAP protocol and MUST be secured.

In particular, communications between the joining node and the Group Manager leverage protocol-specific transport profiles of ACE to achieve communication security, proof-of-possession and server authentication. Note that it is expected that in the commonly referred base-case of this specification, the transport profile to use is pre-configured and well-known to nodes participating in constrained applications.

<!-- 13-01-2020 FP: Covered in ace-key-groupcomm
In particular, communications between the joining node and the Group Manager leverage protocol-specific transport profiles of ACE to achieve communication security, proof-of-possession and server authentication. To this end, the AS MAY signal the specific transport profile to use, consistently with requirements and assumptions defined in the ACE framework {{I-D.ietf-ace-oauth-authz}}. Note that in the commonly referred base-case the transport profile to use is pre-configured and well-known to nodes participating in constrained applications.

With reference to the AS, communications between the joining node and the AS (/token endpoint) as well as between the Group Manager and the AS (/introspect endpoint) can be secured by different means, for instance using DTLS {{RFC6347}} or OSCORE {{RFC8613}}. Further details on how the AS secures communications (with the joining node and the Group Manager) depend on the specifically used transport profile of ACE, and are out of the scope of this specification.
-->

## Overview of the Joining Process {#ssec-overview-join-process}

A node performs the steps described in Section 2 of {{I-D.ietf-ace-key-groupcomm}} in order to join an OSCORE group. The format and processing of messages exchanged among the participants are further specified in {{sec-joining-node-to-AS}} and {{sec-joining-node-to-GM}} of this document. 

<!-- 13-01-2020 FP: Moved to ace-key-groupcomm

A node performs the following steps in order to join an OSCORE group. The format and processing of messages exchanged among the participants follow what is defined in {{I-D.ietf-ace-key-groupcomm}}, and are further specified in {{sec-joining-node-to-AS}} and {{sec-joining-node-to-GM}} of this document. The Group Manager acts as the Key Distribution Center (KDC) defined in {{I-D.ietf-ace-key-groupcomm}}.

1. The joining node requests an Access Token from the AS, in order to access a group-membership resource on the Group Manager and hence join the associated OSCORE group (see {{sec-joining-node-to-AS}}). The joining node will start or continue using a secure communication association with the Group Manager, according to the response from the AS.

2. The joining node transfers authentication and authorization information to the Group Manager, by posting the obtained Access Token to the /authz-info endpoint at the Group Manager (see {{sec-joining-node-to-GM}}). After that, a joining node MUST have a secure communication association established with the Group Manager, before starting to join an OSCORE group under that Group Manager (see {{sec-joining-node-to-GM}}). Possible ways to provide a secure communication association are DTLS {{RFC6347}} and OSCORE {{RFC8613}}.

3. The joining node starts the joining process to become a member of the OSCORE group, by accessing the related group-membership resource hosted by the Group Manager (see {{sec-joining-node-to-GM}}).

4. At the end of the joining process, the joining node has received from the Group Manager the parameters and keying material to securely communicate with the other members of the OSCORE group.

5. The joining node and the Group Manager maintain the secure association, to support possible future communications. These especially include key management operations, such as retrieval of updated keying material from the Group Manager or participation to a group rekeying process (see {{ssec-overview-group-rekeying-process}}).

All further communications between the joining node and the Group Manager MUST be secured, for instance with the same secure association mentioned in step 2.

-->

## Overview of the Group Rekeying Process {#ssec-overview-group-rekeying-process}

If the application requires backward and forward security, the Group Manager MUST generate new security parameters and group keying material, and distribute them to the group (rekeying) upon membership changes.

That is, the group is rekeyed when a node joins the group as a new member, or after a current member leaves the group. By doing so, a joining node cannot access communications in the group prior its joining, while a leaving node cannot access communications in the group after its leaving.

Parameters and group keying material include a new Group Identifier (Gid) for the group and a new Master Secret for the OSCORE Common Security Context of that group (see Section 2 of {{I-D.ietf-core-oscore-groupcomm}}). Once completed a group rekeying, the GM MUST increment the version number of the group keying material.

The Group Manager MUST support the Group Rekeying Process described in {{sec-group-rekeying-process}}. Future application profiles may define alternative message formats and distribution schemes to perform group rekeying.

# Joining Node to Authorization Server {#sec-joining-node-to-AS}

This section describes how the joining node interacts with the AS in order to be authorized to join an OSCORE group under a given Group Manager. In particular, it considers a joining node that intends to contact that Group Manager for the first time.

The message exchange between the joining node and the AS consists of the messages Authorization Request and Authorization Response defined in Section 3 of {{I-D.ietf-ace-key-groupcomm}}. Note that what is defined in {{I-D.ietf-ace-key-groupcomm}} applies, and only additions or modifications to that specification are defined here.


<!-- 13-01-2020 FP: Covered in ace-key-groupcomm

In case the specific AS associated to the Group Manager is unknown to the joining node, the latter can rely on mechanisms like the Unauthorized Resource Request message described in Section 5.1.1 of {{I-D.ietf-ace-oauth-authz}} to discover the correct AS to contact.

-->

## Authorization Request {#ssec-auth-req}

The Authorization Request message defined in Section 3.1 of {{I-D.ietf-ace-key-groupcomm}}, with the following additions:

* The 'scope' parameter MUST be present.

* The identifier of the OSCORE group to join under the Group Manager is encoded as a CBOR text string.

* The role is encoded as a text string. Accepted values of roles are: "requester", "responder", and "monitor". Possible combinations are: \["requester" , "responder"\]; \["requester" , "monitor"\].

* The 'audience' parameter MUST be present.

## Authorization Response {#ssec-auth-resp}

<!-- 13-01-2020 FP: Covered in ace-key-groupcomm

The AS is responsible for authorizing the joining node to join specific OSCORE groups, according to join policies enforced on behalf of the respective Group Manager.

In case of successful authorization, the AS releases an Access Token bound to a proof-of-possession key associated to the joining node.

-->

The Authorization Response message defined in Section 3.2 of {{I-D.ietf-ace-key-groupcomm}}, with the following additions:

* The AS MUST include the 'exp' parameter. Other means for the AS to specify the lifetime of Access Tokens are out of the scope of this specification.

* The AS MUST include the 'scope' parameter, when the value included in the Access Token differs from the one specified by the joining node in the request. In such a case, the second element of 'scope' MUST be present and includes the role or CBOR array of roles that the joining node is actually authorized to take in the group, encoded as specified in {{ssec-auth-req}} of this document.

<!-- 13-01-2020 FP: Covered in ace-key-groupcomm and ace

* The AS MAY also include the 'ace-profile' parameter in the response to the joining node, in order to indicate the specific transport profile of ACE to use for securing communications between the joining node and the Group Manager (see Section 5.6.4.3 of {{I-D.ietf-ace-oauth-authz}}).

In particular, if symmetric keys are used, the AS generates a proof-of-possession key, binds it to the Access Token, and provides it to the joining node in the 'cnf' parameter of the  Access Token response. Instead, if asymmetric keys are used, the joining node provides its own public key to the AS in the 'req_cnf' parameter of the Access Token request. Then, the AS uses it as proof-of-possession key bound to the Access Token, and provides the joining node with the Group Manager's public key in the 'rs_cnf' parameter of the Access Token response.

-->

# Joining a Group {#sec-joining-node-to-GM}

The following subsections describe the interactions between the joining node and the Group Manager, i.e. the sending of the Access Token and the Request-Response exchange to join the OSCORE group. The message exchange between the joining node and the KDC consists of the messages defined in Section 3.3 and 4 of {{I-D.ietf-ace-key-groupcomm}}. Note that what is defined in {{I-D.ietf-ace-key-groupcomm}} applies, and only additions or modifications to that specification are defined here.


## Token Post {#ssec-token-post}

The Token post exchange is defined in Section 3.3 of {{I-D.ietf-ace-key-groupcomm}}. Additionally, if allowed by the used transport profile of ACE, the joining node may instead provide the Access Token to the Group Manager by other means, e.g. during a secure session establishment (see Section 3.3.1 of {{I-D.ietf-ace-dtls-authorize}}).

Additionally to what defined in {{I-D.ietf-ace-key-groupcomm}}, the following applies.

* The 'rsnonce' parameter contains a dedicated 8-byte nonce N_S generated by the Group Manager. The joining node may use this nonce in order to prove the possession of its own private key, upon joining the group (see {{ssec-join-req-sending}}).

* If present in the response:

  * 'sign_alg' takes value from Tables 5 and 6 of {{RFC8152}}.

  * 'sign_parameters' takes values from the "Counter Signature Parameters" Registry (see Section 9.1 of {{I-D.ietf-core-oscore-groupcomm}}). Its structure depends on the value of 'sign_alg'. If no parameters of the counter signature algorithm are specified, 'sign_parameters' MUST be encoding the CBOR simple value Null.

  * 'sign_key_parameters' takes values from the "Counter Signature Key Parameters" Registry (see Section 9.2 of {{I-D.ietf-core-oscore-groupcomm}}). Its structure depends on the value of 'sign_alg'. If no parameters of the key used with the counter signature algorithm are specified, 'sign_key_parameters' MUST be encoding the CBOR simple value Null.

  * 'pub_key_enc' takes value 1 ("COSE\_Key") from the 'Confirmation Key' column of the "CWT Confirmation Method" Registry defined in {{I-D.ietf-ace-cwt-proof-of-possession}}, so indicating that public keys in the OSCORE group are encoded as COSE Keys {{RFC8152}}. Future specifications may define additional values for this parameter.

<!-- 13-01-2020 FP: Covered in ace-key-groupcomm 

At this point in time, the joining node might not have all the necessary information concerning the public keys in the OSCORE group, as well as concerning the algorithm and related parameters for computing countersignatures in the  OSCORE group. In such a case, the joining node MAY use the 'sign_info' and 'pub_key_enc' parameters defined in Section 3.3 of {{I-D.ietf-ace-key-groupcomm}} to ask for such information.

Alternatively, the joining node may retrieve this information by other means, e.g. by using the approach described in {{I-D.tiloca-core-oscore-discovery}}.

If the Access Token is valid, the Group Manager responds to the POST request with a 2.01 (Created) response, according to what is specified in the signalled transport profile of ACE. The Group Manager MUST use the Content-Format "application/ace+cbor" defined in Section 8.14 of {{I-D.ietf-ace-oauth-authz}}.

The payload of the 2.01 (Created) response is a CBOR map, which MUST include the 'rsnonce' parameter defined in Section 3.3.3 of {{I-D.ietf-ace-key-groupcomm}}, and MAY include the 'sign_info' parameter as well as the 'pub_key_enc' parameter, defined in its Sections 3.3.1 and 3.3.2, respectively.  Note that this deviates from the default payload format for this response message as defined in the ACE framework (see Section 5.8.1 of {{I-D.ietf-ace-oauth-authz}}).

The 'rsnonce' parameter includes a dedicated 8-byte nonce N_S generated by the Group Manager. The joining node may use this nonce in order to prove the possession of its own private key, upon joining the group (see {{ssec-join-req-sending}}).

If present in the response:

* 'sign_alg', i.e. the first element of the 'sign_info' parameter, takes value from Tables 5 and 6 of {{RFC8152}}.

* 'sign_parameters', i.e. the second element of the 'sign_info' parameter, takes values from the "Counter Signature Parameters" Registry (see Section 9.1 of {{I-D.ietf-core-oscore-groupcomm}}). Its structure depends on the value of 'sign_alg'. If no parameters of the counter signature algorithm are specified, 'sign_parameters' MUST be encoding the CBOR simple value Null.

* 'sign_key_parameters', i.e. the third element of the 'sign_info' parameter, takes values from the "Counter Signature Key Parameters" Registry (see Section 9.2 of {{I-D.ietf-core-oscore-groupcomm}}). Its structure depends on the value of 'sign_alg'. If no parameters of the key used with the counter signature algorithm are specified, 'sign_key_parameters' MUST be encoding the CBOR simple value Null.

* 'pub_key_enc' takes value 1 ("COSE\_Key") from the 'Confirmation Key' column of the "CWT Confirmation Method" Registry defined in {{I-D.ietf-ace-cwt-proof-of-possession}}, so indicating that public keys in the OSCORE group are encoded as COSE Keys {{RFC8152}}. Future specifications may define additional values for this parameter.

The CBOR map specified as payload of the 2.01 (Created) response may include further parameters, e.g. according to the signalled transport profile of ACE.

Finally, the joining node establishes a secure channel with the Group Manager, according to what is specified in the Access Token response and the signalled transport profile of ACE.

-->

## Sending the Joining Request {#ssec-join-req-sending}

The joining node requests to join the OSCORE group, by sending a Joining Request message to the related group-membership resource at the Group Manager, as per Section 4.2 of {{I-D.ietf-ace-key-groupcomm}}.

Additionally to what defined in {{I-D.ietf-ace-key-groupcomm}}, the following applies.

* The string "group-oscore" is used instead of "ace-group" (see Section 4.1 of {{I-D.ietf-ace-key-groupcomm}}) as the top level path. The url-path /group-oscore/ is a default name of this specifications: implementations are not required to use this name, and can define their own instead.

<!-- 13-01-2020 FP: Covered in ace-key-groupcomm 

In particular, the joining node sends a CoAP POST request to the endpoint /group-oscore/GROUPNAME at the Group Manager, where GROUPNAME is the name of the OSCORE group to join. This Joining Request has content format application/ace-groupcomm+cbor defined in Section 8.1 of {{I-D.ietf-ace-key-groupcomm}}, and is formatted as defined in Section 4.1.2.1 of {{I-D.ietf-ace-key-groupcomm}}. Specifically:

-->

* The 'scope' parameter MUST be present.

* The 'get_pub_keys' parameter is present only if the joining node wants to retrieve the public keys of the group members from the Group Manager during the joining process (see {{sec-public-keys-of-joining-nodes}}). Otherwise, this parameter MUST NOT be present.


<!-- 13-01-2020 FP: Covered in ace-key-groupcomm 
* The 'client_cred' parameter, if present, includes the public key of the joining node. In case the joining node knows the encoding of public keys in the OSCORE group, as well as the countersignature algorithm and possible associated parameters used in the OSCORE group, the included public key MUST be compatible with those criteria. That is, the public key MUST be encoded according to the encoding of public keys in the OSCORE group, and MUST be compatible with the countersignature algorithm and possible associated parameters used in the OSCORE group. This parameter MAY be omitted if: i) the joining node is asking to access the group exclusively as monitor; or ii) the Group Manager already acquired this information, for instance during a past joining process. In any other case, this parameter MUST be present.

Furthermore, if the 'client_cred' parameter is present, the CBOR map specified as payload of the Joining Request MUST also include the following parameters.
-->
* 'cnonce' contains a dedicated 8-byte nonce N_C generated by the Client.

* The signature encoded in the 'client_cred_verify' parameter is computed by the joining node by using the same private key and countersignature algorithm it intends to use for signing messages in the OSCORE group. Moreover, N_S is defined as defined in {{sssec-challenge-value}}.

### Value of the N\_S Challenge {#sssec-challenge-value}

The N\_S challenge takes one of the following values.

1. If the joining node has posted the Access Token to the /authz-info endpoint of the Group Manager as in {{ssec-token-post}}, N\_S takes the same value of the 'rsnonce' parameter in the 2.01 (Created) response to the Token POST.

2. If the Token POST in {{ssec-token-post}} has relied on the DTLS profile of ACE {{I-D.ietf-ace-dtls-authorize}} and the joining node included the Access Token as content of the "psk_identity" field of the ClientKeyExchange message {{RFC6347}}, N\_S is an exporter value computed as defined in Section 7.5 of {{RFC8446}}. Specifically, N\_S is exported from the DTLS session between the joining node and the Group Manager, using an empty 'context_value', 32 bytes as 'key_length', and the exporter label "EXPORTER-ACE-Sign-Challenge" defined in Section 7 of {{I-D.ietf-ace-mqtt-tls-profile}}.

3. If the joining node is in fact re-joining the group, without posting again the same and still valid Access Token:

   - If the joining node and the Group Manager communicates over DTLS, N\_S is an exporter value, computed as described in point (2) above.
   
   - If the joining node and the Group Manager communicates over OSCORE, the
     N\_S is the output PRK of a HKDF-Extract step {{RFC5869}}, i.e. PRK = HMAC-Hash(salt, IKM).  In particular, 'salt' takes (x1 | x2), where x1 is the ID Context of the OSCORE Security Context between the joining node and the Group Manager, x2 is the Sender ID of the joining node in that Context, and | denotes byte string concatenation.  Also, 'IKM' is the OSCORE Master Secret of the OSCORE Security Context between the joining node and the Group Manager. The HKDF MUST be one of the HMAC-based HKDF {{RFC5869}} algorithms defined for COSE {{RFC8152}}. HKDF SHA-256 is mandatory to implement.
     
It is up to applications to define how N_S is computed in further alternative settings.

## Processing the Joining Request {#ssec-join-req-processing}

The Group Manager processes the Joining Request as defined in Section 4.1.2.1 of {{I-D.ietf-ace-key-groupcomm}}. 
Additionally, the following applies.

* The Group Manager MUST return a 4.00 (Bad Request) response in case the Joining Request includes the 'client_cred' parameter but does not include both the 'cnonce' and 'client_cred_verify' parameters.

* In case the Joining Request does not include the 'client_cred' parameter, the joining process fails if the Group Manager either: i) does not store a public key with an accepted format for the joining node; or ii) stores multiple public keys with an accepted format for the joining node.

* To compute the signature contained in 'client_cred_verify', the GM considers: i) as signed value, N_S concatenated with N_C, where N_S is determined as described in {{sssec-challenge-value}}, while N_C is the nonce provided in the 'cnonce' parameter of the Joining Request; ii) the countersignature algorithm used in the OSCORE group, and possible correponding parameters; and iii) the public key of the joining node, either retrieved from the 'client_cred' parameter, or already stored as acquired from previous interactions with the joining node. 

* The payload of a 4.00 Bad Request response from the GM to the Client MUST contain the 'sign_info' and the 'pub_key_enc' parameters.

* When receiving a 4.00 Bad Request response, the joining node SHOULD send a new Joining Request to the Group Manager, containing:

  * The 'client_cred' parameter, including a public key compatible with the encoding, countersignature algorithm and possible associated parameters indicated by the Group Manager.

  * The 'client_cred_verify' parameter, including a signature computed as described in {{ssec-join-req-sending}}, by using the public key indicated in the current 'client_cred' parameter, with the countersignature algorithm and possible associated parameters indicated by the Group Manager.


<!-- 13-01-2020 FP: Covered in ace-key-groupcomm - added this text in ace key groupcomm, was definitely missing 

* In case the Joining Request includes the 'client_cred' parameter, the Group Manager checks that the public key of the joining node has an accepted format. That is, the public key has to be encoded as expected in the OSCORE group, and has to be compatible with the counter signature algorithm and possible associated parameters used in the OSCORE group. The joining process fails if the public key of the joining node does not have an accepted format.

* In case the Joining Request does not include the 'client_cred' parameter, the Group Manager checks whether it is storing a public key for the joining node, which is compatible with the encoding, counter signature algorithm and possible associated parameters used in the OSCORE group. The joining process fails if the Group Manager either: i) does not store a public key with an accepted format for the joining node; or ii) stores multiple public keys with an accepted format for the joining node.

* In case the Joining Request includes the 'client_cred_verify' parameter, the Group Manager verifies the signature contained in the parameter. To this end, it considers: i) as signed value, N_S concatenated with N_C, where N_S is determined as described in {{sssec-challenge-value}}, while N_C is the nonce provided in the 'cnonce' parameter of the Joining Request; ii) the countersignature algorithm used in the OSCORE group, and possible correponding parameters; and iii) the public key of the joining node, either retrieved from the 'client_cred' parameter, or already stored as acquired from previous interactions with the joining node. The joining process fails if the Group Manager does not successfully verify the signature.

If the joining process has failed, the Group Manager MUST reply to the joining node with a 4.00 (Bad Request) response. The payload of this response is a CBOR map, which includes a 'sign_info' parameter and a 'pub_key_enc' parameter, formatted as in the Token POST response in {{ssec-token-post}}.

Upon receiving this response, the joining node SHOULD send a new Joining Request to the Group Manager, which contains:

* The 'client_cred' parameter, including a public key compatible with the encoding, countersignature algorithm and possible associated parameters indicated by the Group Manager.

* The 'client_cred_verify' parameter, including a signature computed as described in {{ssec-join-req-sending}}, by using the public key indicated in the current 'client_cred' parameter, with the countersignature algorithm and possible associated parameters indicated by the Group Manager.

-->

## Joining Response {#ssec-join-resp}

If the processing of the Joining Request described in {{ssec-join-req-processing}} is successful, the Group Manager updates the group membership by registering the joining node NODENAME as a new member of the OSCORE group GROUPNAME, as described in {{I-D.ietf-ace-key-groupcomm}}.


<!-- 13-01-2020 FP: Covered in ace-key-groupcomm 

In particular, the Group Manager generates a node name NODENAME associated to the joining node, ensuring that the name is unique within that OSCORE group. Then, the Group Manager creates a new node resource, as sub-resource of the group-membership resource associated to the OSCORE group, for instance at /group-oscore/GROUPNAME/NODENAME .

-->

If the joining node is not exclusively configured as monitor, the Group Manager performs also the following actions.

* The Group Manager selects an available OSCORE Sender ID in the OSCORE group, and exclusively assigns it to the joining node.


<!-- 13-01-2020 FP: Covered in ace-key-groupcomm -

* If the 'client_cred' parameter was present in the request, the Group Manager adds the specified public key of the joining node to the list of public keys of the current group members. 

* If the 'client_cred' parameter was not present in the request, the Group Manager retrieves the already stored public key of the joining node, as acquired from previous interactions (see also {{sec-public-keys-of-joining-nodes}}). Then, the Group Manager adds the retrieved public key to the list of public keys of the current group members.

-->

* The Group Manager stores the association between i) the public key of the joining node; and ii) the Group Identifier (Gid) associated to the OSCORE group together with the OSCORE Sender ID assigned to the joining node in the group. The Group Manager MUST keep this association updated over time.

Then, the Group Manager replies to the joining node, providing the updated security parameters and keying meterial necessary to participate in the group communication. This success Joining Response is formatted as defined in Section 4.1.2.1 of {{I-D.ietf-ace-key-groupcomm}}, with the following additions:

* The 'gkty' parameter identifies a key of type "Group_OSCORE_Security_Context object", defined in {{ssec-iana-groupcomm-key-registry}} of this specification.

* The 'key' parameter includes what the joining node needs in order to set up the OSCORE Security Context as per Section 2 of {{I-D.ietf-core-oscore-groupcomm}}. This parameter has as value a Group_OSCORE_Security_Context object, which is defined in this specification and extends the OSCORE_Security_Context object encoded in CBOR as defined in Section 3.2.1 of {{I-D.ietf-ace-oscore-profile}}. In particular, it contains the additional parameters 'cs_alg', 'cs_params',  'cs_key_params' and 'cs_key_enc' defined in {{ssec-iana-security-context-parameter-registry}} of this specification. More specifically, the 'key' parameter is composed as follows.

   * The 'ms' parameter MUST be present and includes the OSCORE Master Secret value.

   * The 'clientId' parameter, if present, has as value the OSCORE Sender ID assigned to the joining node by the Group Manager, as described above. This parameter is not present if the node joins the group exclusively as monitor, according to what specified in the Access Token (see {{ssec-auth-resp}}). In any other case, this parameter MUST be present.

   * The 'hkdf' parameter, if present, has as value the KDF algorithm used in the group.

   * The 'alg' parameter, if present, has as value the AEAD algorithm used in the group.

   * The 'salt' parameter, if present, has as value the OSCORE Master Salt.

   * The 'contextId' parameter MUST be present and has as value the Group Identifier (Gid) associated to the OSCORE group.

   * The 'rpl' parameter, if present, specifies the OSCORE Replay Window Size and Type value.

   * The 'cs_alg' parameter MUST be present and specifies the algorithm used to countersign messages in the group. This parameter takes values from Tables 5 and 6 of {{RFC8152}}.

   * The 'cs_params' parameter MAY be present and specifies the additional parameters for the counter signature algorithm. This parameter is a CBOR map whose content depends on the counter signature algorithm, as specified in Section 2 and Section 9.1 of {{I-D.ietf-core-oscore-groupcomm}}.

   * The 'cs_key_params' parameter MAY be present and specifies the additional parameters for the key used with the counter signature algorithm. This parameter is a CBOR map whose content depends on the counter signature algorithm, as specified in Section 2 and Section 9.2 of {{I-D.ietf-core-oscore-groupcomm}}.

  * The 'cs_key_enc' parameter MAY be present and specifies the encoding of the public keys of the group members. This parameter is a CBOR integer, whose value is 1 ("COSE\_Key") taken from the 'Confirmation Key' column of the "CWT Confirmation Method" Registry defined in {{I-D.ietf-ace-cwt-proof-of-possession}}, so indicating that public keys in the OSCORE group are encoded as COSE Keys {{RFC8152}}. Future specifications may define additional values for this parameter. If this parameter is not present, 1 ("COSE\_Key") MUST be assumed as default value.

* The 'num' parameter MUST be present.
 
* The 'ace-groupcomm-profile' parameter MUST be present and has value coap_group_oscore_app (TBD), which is defined in {{ssec-iana-groupcomm-profile-registry}} of this specification.

* The 'exp' parameter MUST be present.

* The 'pub_keys', if present, includes the public keys of the group members that are relevant to the joining node. That is, it includes: i) the public keys of the responders currently in the group, in case the joining node is configured (also) as requester; and ii) the public keys of the requesters currently in the group, in case the joining node is configured (also) as responder or monitor. If public keys are encoded as COSE\_Keys, each of them has as 'kid' the Sender ID that the corresponding owner has in the group, thus used as group member identifier.

* The 'group_policies' parameter SHOULD be present.

<!-- 13-01-2020 FP: Covered in ace-key-groupcomm -

and includes a list of parameters indicating particular policies enforced in the group. In particular, if the field "Sequence Number Synchronization Method" is present, it indicates the method to achieve synchronization of sequence numbers among group members (see Appendix E of {{I-D.ietf-core-oscore-groupcomm}}), by specifying the corresponding value from the "Sequence Number Synchronization Method" Registry defined in Section 8.7 of {{I-D.ietf-ace-key-groupcomm}}.

Finally, the joining node uses the information received in the Joining Response to set up the OSCORE Security Context, as described in Section 2 of {{I-D.ietf-core-oscore-groupcomm}}. From then on, the joining node can exchange group messages secured with Group OSCORE as described in {{I-D.ietf-core-oscore-groupcomm}}.
-->

The rekeying process in case backward security is needed is described in {{sec-group-rekeying-process}}.

# Public Keys of Joining Nodes # {#sec-public-keys-of-joining-nodes}

Source authentication of OSCORE messages exchanged within the group is ensured by means of digital counter signatures (see Sections 2 and 3 of {{I-D.ietf-core-oscore-groupcomm}}). Therefore, group members must be able to retrieve each other's public key from a trusted key repository, in order to verify source authenticity of incoming group messages.

As also discussed in {{I-D.ietf-core-oscore-groupcomm}}, the Group Manager acts as trusted repository of the public keys of the group members, and provides those public keys to group members if requested to. Upon joining an OSCORE group, a joining node is thus expected to provide its own public key to the Group Manager.

In particular, one of the following four cases can occur when a new node joins an OSCORE group.

* The joining node is going to join the group exclusively as monitor. That is, it is not going to send messages to the group, and hence to produce signatures with its own private key. In this case, the joining node is not required to provide its own public key to the Group Manager, which thus does not have to perform any check related to the public key encoding, or to a countersignature algorithm and possible associated parameters for that joining node.

* The Group Manager already acquired the public key of the joining node during a past joining process. In this case, the joining node MAY choose not to provide again its own public key to the Group Manager, in order to limit the size of the Joining Request. The joining node MUST provide its own public key again if it has provided the Group Manager with multiple public keys during past joining processes, intended for different OSCORE groups. If the joining node provides its own public key, the Group Manager performs consistency checks as per {{ssec-join-resp}} and, in case of success, considers it as the public key associated to the joining node in the OSCORE group.

* The joining node and the Group Manager use an asymmetric proof-of-possession key to establish a secure communication channel. Then, two cases can occur.

   1. The proof-of-possession key is compatible with the encoding as well as with the counter signature algorithm and possible associated parameters used in the OSCORE group. Then, the Group Manager considers the proof-of-possession key as the public key associated to the joining node in the OSCORE group. If the joining node is aware that the proof-of-possession key is also valid for the OSCORE group, it MAY not provide it again as its own public key to the Group Manager. The joining node MUST provide its own public key again if it has provided the Group Manager with multiple public keys during past joining processes, intended for different OSCORE groups. If the joining node provides its own public key in the 'client_cred' parameter of the Joining Request (see {{ssec-join-req-sending}}), the Group Manager performs consistency checks as per {{ssec-join-resp}} and, in case of success, considers it as the public key associated to the joining node in the OSCORE group.

   2. The proof-of-possession key is not compatible with the encoding or with the counter signature algorithm and possible associated parameters used in the OSCORE group. In this case, the joining node MUST provide a different compatible public key to the Group Manager in the 'client_cred' parameter of the Joining Request (see {{ssec-join-req-sending}}). Then, the Group Manager performs consistency checks on this latest provided public key as per {{ssec-join-resp}} and, in case of success, considers it as the public key associated to the joining node in the OSCORE group.

* The joining node and the Group Manager use a symmetric proof-of-possession key to establish a secure communication channel. In this case, upon performing a joining process with that Group Manager for the first time, the joining node specifies its own public key in the 'client_cred' parameter of the Joining Request targeting the group-membership endpoint (see {{ssec-join-req-sending}}).

# Retrieval of Updated Keying Material # {#sec-updated-key}

At some point, a group member considers the OSCORE Security Context invalid and to be renewed. This happens, for instance, after a number of unsuccessful security processing of incoming messages from other group members, or when the Security Context expires as specified by the 'exp' parameter of the Joining Response. 

When this happens, the group member retrieves updated security parameters and group keying material. This can occur in the two different ways described below.

## Retrieval of Group Keying Material ## {#ssec-updated-key-only}

If the group member wants to retrieve only the latest group keying material, it sends a Key Distribution Request to the Group Manager.

In particular, it sends a CoAP GET request to the endpoint /group-oscore/GROUPNAME at the Group Manager.

The Group Manager processes the Key Distribution Request according to Section 4.1.2.2 of {{I-D.ietf-ace-key-groupcomm}}. The Key Distribution Response is formatted as defined in Section 4.1.2.2 of {{I-D.ietf-ace-key-groupcomm}}.

Upon receiving the Key Distribution Response, the group member retrieves the updated security parameters and group keying material, and, if they differ from the current ones, use them to set up the new OSCORE Security Context as described in Section 2 of {{I-D.ietf-core-oscore-groupcomm}}.

## Retrieval of Group Keying Material and Sender ID ## {#ssec-updated-and-individual-key}

If the group member wants to retrieve the latest group keying material as well as the Sender ID that it has in the OSCORE group, it sends a Key Distribution Request to the Group Manager.

In particular, it sends a CoAP GET request to the endpoint /group-oscore/GROUPNAME/NODENAME at the Group Manager.

The Group Manager processes the Key Distribution Request according to Section 4.1.6.2 of {{I-D.ietf-ace-key-groupcomm}}. The Key Distribution Response is formatted as defined in Section 4.1.2.2 of {{I-D.ietf-ace-key-groupcomm}}.

Upon receiving the Key Distribution Response, the group member retrieves the updated security parameters, group keying material and Sender ID, and, if they differ from the current ones, use them to set up the new OSCORE Security Context as described in Section 2 of {{I-D.ietf-core-oscore-groupcomm}}.

# Retrieval of New Keying Material # {#sec-new-key}

As discussed in Section 2.2 of {{I-D.ietf-core-oscore-groupcomm}}, a group member may at some point experience a wrap-around of its own Sender Sequence Number in the group.

When this happens, the group member MUST send a Key Renewal Request message to the Group Manager, as per Section 4.4 of {{I-D.ietf-ace-key-groupcomm}}. In particular, it sends a CoAP PUT request to the endpoint /group-oscore/GROUPNAME/NODENAME at the Group Manager.

Upon receiving the Key Renewal Request, the Group Manager processes it as defined in Section 4.1.6.2 of {{I-D.ietf-ace-key-groupcomm}}, and performs one of the following actions.

1. The Group Manager replies to the group member with a 4.06 (Not Acceptable) error response, and rekeys the whole OSCORE group as discussed in {{sec-group-rekeying-process}}.

2. The Group Manager generates a new Sender ID for that group member and replies with a Key Renewal Response, formatted as defined in Section 4.1.6.2 of {{I-D.ietf-ace-key-groupcomm}}. In particular, the CBOR Map in the response payload includes a single parameter 'clientId' defined in {{ssec-iana-ace-groupcomm-parameters-registry}} of this document, specifying the new Sender ID of the group member encoded as a CBOR byte string.

# Retrieval of Public Keys of Group Members # {#sec-pub-keys}

A group member may need to retrieve the public keys of other group members. To this end, the group member sends a Public Key Request message to the Group Manager, as per Section 4.5 of {{I-D.ietf-ace-key-groupcomm}}. In particular, it sends the request to the endpoint /group-oscore/GROUPNAME/pub-key at the Group Manager.

If the Public Key Request uses the method FETCH, the Public Key Request is formatted as defined in Section 4.1.3.1 of {{I-D.ietf-ace-key-groupcomm}}. In particular, each element of the 'get_pub_keys' parameter is a CBOR byte string, which encodes the Sender ID of the group member for which the associated public key is requested.

Upon receiving the Public Key Request, the Group Manager processes it as per Section 4.1.3.1 or 4.1.3.2 of {{I-D.ietf-ace-key-groupcomm}}, depending on the request method being FETCH or GET, respectively. Additionally, if the Public Key Request uses the method FETCH, the Group Manager silently ignores identifiers included in the ’get_pub_keys’ parameter of the request that are not associated to any current group member.

The success Public Key Response is formatted as defined in Section 4.1.3.1 of {{I-D.ietf-ace-key-groupcomm}}.

# Retrieval of Group Policies # {#sec-policies}

A group member may request the current policies used in the OSCORE group. To this end, the group member sends a Policies Request, as per Section 4.6  of {{I-D.ietf-ace-key-groupcomm}}. In particular, it sends a CoAP GET request to the endpoint /group-oscore/GROUPNAME/policies at the Group Manager, where GROUPNAME is the name of the OSCORE group.

Upon receiving the Policies Request, the Group Manager processes it as per Section 4.1.4.1 of {{I-D.ietf-ace-key-groupcomm}}. The success Policies Response is formatted as defined in Section 4.1.4.1 of {{I-D.ietf-ace-key-groupcomm}}.

# Retrieval of Keying Material Version # {#sec-version}

A group member may request the current version of the keying material used in the OSCORE group. To this end, the group member sends a Version Request, as per Section 4.7 of {{I-D.ietf-ace-key-groupcomm}}. In particular, it sends a CoAP GET request to the endpoint /group-oscore/GROUPNAME/ctx-num at the Group Manager, where GROUPNAME is the name of the OSCORE group.

Upon receiving the Version Request, the Group Manager processes it as per Section 4.1.5.1 of {{I-D.ietf-ace-key-groupcomm}}. The success Version Response is formatted as defined in Section 4.1.5.1 of {{I-D.ietf-ace-key-groupcomm}}.

# Request to Leave the Group # {#sec-leave-req}

A group member may request to leave the OSCORE group. To this end, the group member sends a Group Leaving Request, as per Section 4.8 of {{I-D.ietf-ace-key-groupcomm}}. In particular, it sends a CoAP DELETE request to the endpoint /group-oscore/GROUPNAME/NODENAME at the Group Manager.

Upon receiving the Leaving Request, the Group Manager processes it as per Section 4.1.6.3 of {{I-D.ietf-ace-key-groupcomm}}.

# Removal of a Group Member # {#sec-leaving}

Other than after a spontaneous request to the Group Manager as described in {{sec-leave-req}}, a node may be forcibly removed from the OSCORE group, e.g. due to expired or revoked authorization.

In either case, if the leaving node is not configured exclusively as monitor, the Group Manager performs the following actions.

* The Group Manager frees the OSCORE Sender ID value of the leaving node, which becomes available for possible upcoming joining nodes.

* The Group Manager cancels the association between, on one hand, the public key of the leaving node and, on the other hand, the Group Identifier (Gid) associated to the OSCORE group together with the freed OSCORE Sender ID value. The Group Manager deletes the public key of the leaving node, if that public key has no remaining association with any pair (Group ID, Sender ID).

If the application requires forward security, the Group Manager MUST generate updated security parameters and group keying material, and provide it to the remaining group members (see {{sec-group-rekeying-process}}). As a consequence, the leaving node is not able to acquire the new security parameters and group keying material distributed after its leaving.

Same considerations in Section 5 of {{I-D.ietf-ace-key-groupcomm}} apply here as well, considering the Group Manager acting as KDC.

# Group Rekeying Process {#sec-group-rekeying-process}

In order to rekey the OSCORE group, the Group Manager distributes a new Group ID of the group and a new OSCORE Master Secret for that group. When doing so, the Group Manager MUST increment the version number of the group keying material. Also, the Group Manager MUST preserve the same unchanged Sender IDs for all group members. This avoids affecting the retrieval of public keys from the Group Manager as well as the verification of message countersignatures.

The Group Manager MUST support at least the following group rekeying scheme. Future application profiles may define alternative message formats and distribution schemes.

The Group Manager uses the same format of the Joining Response message in {{ssec-join-resp}}. In particular:

* Only the parameters 'gkty', 'key', 'num', 'ace-groupcomm-profile' and 'exp' are present.

* The 'ms' parameter of the 'key' parameter specifies the new OSCORE Master Secret value.

* The 'contextId' parameter of the 'key' parameter specifies the new Group ID.

The Group Manager separately sends a group rekeying message to each group member to be rekeyed. Each rekeying message MUST be secured with the pairwise secure communication channel between the Group Manager and the group member used during the joining process.

This approach requires group members to act (also) as servers, in order to correctly handle unsolicited group rekeying messages from the Group Manager. In particular, if a group member and the Group Manager use OSCORE {{RFC8613}} to secure their pairwise communications, the group member MUST create a Replay Window in its own Recipient Context upon establishing the OSCORE Security Context with the Group Manager, e.g. by means of the OSCORE profile of ACE {{I-D.ietf-ace-oscore-profile}}.

Group members and the Group Manager SHOULD additionally support alternative rekeying approaches that do not require group members to act (also) as servers. A number of such approaches are defined in Section 4 of {{I-D.ietf-ace-key-groupcomm}}. In particular, a group member may subscribe for updates to the group-membership resource of the group, at the endpoint /group-oscore/GROUPNAME of the Group Manager, where GROUPNAME is the name of the OSCORE group. This can rely on CoAP Observe {{RFC7641}} or on a full-fledged Pub-Sub model {{I-D.ietf-core-coap-pubsub}} with the Group Manager acting as Broker.

# Security Considerations {#sec-security-considerations}

The method described in this document leverages the following management aspects related to OSCORE groups and discussed in the sections of {{I-D.ietf-core-oscore-groupcomm}} referred below.

* Management of group keying material (see Section 2.1 of {{I-D.ietf-core-oscore-groupcomm}}). The Group Manager is responsible for the renewal and re-distribution of the keying material in the groups of its competence (rekeying). According to the specific application requirements, this can include rekeying the group upon changes in its membership. In particular, renewing the group keying material is required upon a new node's joining or a current node's leaving, in case backward security and forward security have to be preserved, respectively.

* Provisioning and retrieval of public keys (see Section 2 of {{I-D.ietf-core-oscore-groupcomm}}). The Group Manager acts as key repository of public keys of group members, and provides them upon request.

* Synchronization of sequence numbers (see Section 5.1 of {{I-D.ietf-core-oscore-groupcomm}}). This concerns how a responder node that has just joined an OSCORE group can synchronize with the sequence number of requesters in the same group.

Before sending the Joining Response, the Group Manager MUST verify that the joining node actually owns the associated private key. To this end, the Group Manager can rely on the proof-of-possession challenge-response defined in {{sec-joining-node-to-GM}}. Alternatively, the joining node can use its own public key as asymmetric proof-of-possession key to establish a secure channel with the Group Manager, e.g. as in Section 3.2 of {{I-D.ietf-ace-dtls-authorize}}. However, this requires such proof-of-possession key to be compatible with the encoding as well as with the countersignature algorithm and possible associated parameters used in the OSCORE group.

A node may have joined multiple OSCORE groups under different non-synchronized Group Managers. Therefore, it can happen that those OSCORE groups have the same Group Identifier (Gid). It follows that, upon receiving a Group OSCORE message addressed to one of those groups, the node would have multiple Security Contexts matching with the Gid in the incoming message. It is up to the application to decide how to handle such collisions of Group Identifiers, e.g. by trying to process the incoming message using one Security Context at the time until the right one is found.

Further security considerations are inherited from {{I-D.ietf-ace-key-groupcomm}}, the ACE framework for Authentication and Authorization {{I-D.ietf-ace-oauth-authz}}, and the specific transport profile of ACE signalled by the AS, such as {{I-D.ietf-ace-dtls-authorize}} and {{I-D.ietf-ace-oscore-profile}}.

# IANA Considerations {#sec-iana}

Note to RFC Editor: Please replace all occurrences of "\[\[This specification\]\]" with the RFC number of this specification and delete this paragraph.

This document has the following actions for IANA.

## ACE Groupcomm Key Registry {#ssec-iana-groupcomm-key-registry}

IANA is asked to register the following entry in the "ACE Groupcomm Key" Registry defined in Section 8.4 of {{I-D.ietf-ace-key-groupcomm}}.

*  Name: Group_OSCORE_Security_Context object
*  Key Type Value: TBD
*  Profile: "coap_group_oscore_app", defined in {{ssec-iana-groupcomm-profile-registry}} of this specification.
*  Description: A Group_OSCORE_Security_Context object encoded as described in {{ssec-join-resp}} of this specification.
*  Reference: \[\[This specification\]\]

## OSCORE Security Context Parameters Registry {#ssec-iana-security-context-parameter-registry}

IANA is asked to register the following entries in the "OSCORE Security Context Parameters" Registry defined in Section 9.2 of {{I-D.ietf-ace-oscore-profile}}.

*  Name: cs_alg
*  CBOR Label: TBD
*  CBOR Type: tstr / int
*  Registry: COSE Algorithm Values (ECDSA, EdDSA)
*  Description: OSCORE Counter Signature Algorithm Value
*  Reference: \[\[This specification\]\]

*  Name: cs_params
*  CBOR Label: TBD
*  CBOR Type: map
*  Registry: Counter Signatures Parameters
*  Description: OSCORE Counter Signature Algorithm Additional Parameters
*  Reference: \[\[This specification\]\]

*  Name: cs_key_params
*  CBOR Label: TBD
*  CBOR Type: map
*  Registry: Counter Signatures Key Parameters
*  Description: OSCORE Counter Signature Key Additional Parameters
*  Reference: \[\[This specification\]\]

*  Name: cs_key_enc
*  CBOR Label: TBD
*  CBOR Type: integer
*  Registry: ACE Public Key Encoding
*  Description: Encoding of Public Keys to be used with the OSCORE Counter Signature Algorithm
*  Reference: \[\[This specification\]\]

## ACE Groupcomm Profile Registry {#ssec-iana-groupcomm-profile-registry}

IANA is asked to register the following entry in the "ACE Groupcomm Profile" Registry defined in Section 8.5 of {{I-D.ietf-ace-key-groupcomm}}.

*  Name: coap_group_oscore_app
*  Description: Application profile to provision keying material for participating in group communication protected with Group OSCORE as per {{I-D.ietf-core-oscore-groupcomm}}.
*  CBOR Value: TBD
*  Reference: \[\[This specification\]\]

## Sequence Number Synchronization Method Registry {#ssec-iana-sn-synch-method-registry}

IANA is asked to register the following entries in the "Sequence Number Synchronization Method" Registry defined in Section 8.7 of {{I-D.ietf-ace-key-groupcomm}}.

*  Name: Best effort
*  Value: 1
*  Description: No action is taken.
*  Reference: {{I-D.ietf-core-oscore-groupcomm}} (Appendix E.1).

*  Name: Baseline
*  Value: 2
*  Description: The first received request sets the baseline reference point, and is discarded with no delivery to the application.
*  Reference: {{I-D.ietf-core-oscore-groupcomm}} (Appendix E.2).

*  Name: Echo challenge-response
*  Value: 3
*  Description: Challenge response using the Echo Option for CoAP from {{I-D.ietf-core-echo-request-tag}}.
*  Reference: {{I-D.ietf-core-oscore-groupcomm}} (Appendix E.3).

##  ACE Groupcomm Parameters Registry {#ssec-iana-ace-groupcomm-parameters-registry}

IANA is asked to register the following entry in the "ACE Groupcomm Parameters" Registry defined in Section 8.3 of {{I-D.ietf-ace-key-groupcomm}}.

* Name: clientId
* CBOR Key: TBD
* CBOR Type: Byte string
* Reference: \[\[This document\]\] ({{sec-new-key}}).

--- back

# Profile Requirements # {#profile-req}

TODO: update

This appendix lists the specifications on this application profile of ACE, based on the requiremens defined in Appendix A of {{I-D.ietf-ace-key-groupcomm}}.

* REQ1 - Specify the encoding and value of the identifier of group of 'scope': see {{ssec-auth-req}}.

* REQ2 - Specify the encoding and value of the identifier of roles of 'scope': see {{ssec-auth-req}}.

* REQ3 (Optional) - Specify the acceptable values for 'sign\_alg': values from Tables 5 and 6 of {{RFC8152}}.

* REQ4 (Optional) - Specify the acceptable values for 'sign\_parameters': values from the "Counter Signature Parameters" Registry (see Section 9.1 of {{I-D.ietf-core-oscore-groupcomm}}).

* REQ5 (Optional) - Specify the acceptable values for 'sign\_key\_parameters': values from the "Counter Signature Key Parameters" Registry (see Section 9.2 of {{I-D.ietf-core-oscore-groupcomm}}).

* REQ6 (Optional) - Specify the acceptable values for 'pub\_key\_enc': 1 ("COSE\_Key") from the 'Confirmation Key' column of the "CWT Confirmation Method" Registry defined in {{I-D.ietf-ace-cwt-proof-of-possession}}. Future specifications may define additional values for this parameter.

* REQ7 - Format of the 'key' value: see {{ssec-join-resp}}.

* REQ8 - Acceptable values of 'gkty': Group_OSCORE_Security_Context object (see {{ssec-join-resp}}).

* REQ9: Specify the format of the identifiers of group members: see {{ssec-join-resp}} and {{sec-pub-keys}}.

* REQ10 (Optional) - Specify the format and content of 'group\_policies' entries: three values are defined and registered, as content of the entry "Sequence Number Synchronization Method" (see {{ssec-iana-sn-synch-method-registry}}).

* REQ11 - Communication protocol that the members of the group must use: CoAP, possibly over IP multicast.

* REQ12 - Security protocols that the group members must use to protect their communication: Group OSCORE.

* REQ13 - Profile identifier: coap_group_oscore_app

* REQ14 (Optional) - Specify the encoding of public keys, of 'client\_cred', and of 'pub\_keys' if COSE_Keys are not used: no.

* REQ15 - Specify policies at the KDC to handle member ids that are not included in 'get_pub_keys': see {{sec-pub-keys}}.

* REQ16 - Specify the format and content of 'group\_policies': see {{ssec-join-resp}}.

* REQ17 - Specify the format of newly-generated individual keying material for group members, or of the information to derive it, and corresponding CBOR label: see {{sec-new-key}}.

* REQ18 - Specify how the communication is secured between the Client and KDC: by means of any transport profile of ACE {{I-D.ietf-ace-oauth-authz}} between Client and Group Manager that complies with the requirements in Appendix C of {{I-D.ietf-ace-oauth-authz}}.

* REQ19: Specify how the nonce N\_S is generated, if the token is not being posted (e.g. if it is used directly to validate TLS instead): see {{sssec-challenge-value}}.

* REQ20: Specify if 'mgt_key_material' used, and if yes specify its format and content: not used in this version of the profile.

* OPT1 (Optional) - Specify the encoding of public keys, of 'client\_cred', and of 'pub\_keys' if COSE_Keys are not used: no.

* OPT2 (Optional) - Specify the negotiation of parameter values for signature algorithm and signature keys, if 'sign_info' and 'pub_key_enc' are not used: possible early discovery by using the approach based on the CoRE Resource Directory described in {{I-D.tiloca-core-oscore-discovery}}.

* OPT3 (Optional) - Specify the format and content of 'mgt\_key\_material', if the default is not used: no.

* OPT4 (Optional) - Specify policies that instruct clients to retain unsuccessfully decrypted messages and for how long, so that they can be decrypted after getting updated keying material: no.

# Document Updates # {#sec-document-updates}

RFC EDITOR: PLEASE REMOVE THIS SECTION.

## Version -03 to -04 ## {#sec-03-04}

* New abstract.

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
