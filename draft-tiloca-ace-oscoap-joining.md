---
title: Joining of OSCORE groups in ACE
abbrev: OSCORE group joining in ACE
docname: draft-tiloca-ace-oscoap-joining-03
# date: 2017-04-25
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
    org: RISE SICS AB
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

normative:
  RFC2119:
  RFC7252:
  RFC8174:
  I-D.ietf-core-object-security:
  I-D.ietf-core-oscore-groupcomm:
  I-D.ietf-ace-actors:
  I-D.ietf-ace-oauth-authz:
  I-D.ietf-ace-oscore-profile:
  I-D.ietf-ace-dtls-authorize:

informative:
  I-D.ietf-core-resource-directory:
  RFC4301:
  RFC6347:
  RFC6749:
  RFC7228:
  RFC7231:
  RFC7296:
  RFC7390:
  RFC8152:

--- abstract

This document describes a method to join a group where communications are based on CoAP and secured with Object Security for Constrained RESTful Environments (OSCORE). The proposed method delegates the authentication and authorization of client nodes that join an OSCORE group through a Group Manager server. This approach builds on the ACE framework for Authentication and Authorization, and leverages protocol-specific profiles of ACE to achieve communication security, proof-of-possession and server authentication.

--- middle

# Introduction {#sec-introduction}

Object Security for Constrained RESTful Environments (OSCORE) {{I-D.ietf-core-object-security}} is a method for application-layer protection of the Constrained Application Protocol (CoAP) {{RFC7252}}, using CBOR Object Signing and Encryption (COSE) {{RFC8152}} and enabling end-to-end security of CoAP payload and options.

As described in {{I-D.ietf-core-oscore-groupcomm}}, OSCORE may be used also to protect CoAP group communication over IP multicast {{RFC7390}}. This relies on a Group Manager entity, which is responsible for managing an OSCORE group, where members exchange CoAP messages secured with OSCORE. In particular, the Group Manager coordinates the join process of new group members and can be responsible for multiple groups.

This specification builds on the ACE framework for Authentication and Authorization {{I-D.ietf-ace-oauth-authz}} and defines how a client joins an OSCORE group through a resource server acting as Group Manager. The client acting as joining node relies on an Access Token, which is bound to a proof-of-possession key and authorizes the access to specific join resources at the Group Manager. Exchanged messages follow the formats defined in [NEW_DRAFT].

In order to achieve communication security, proof-of-possession and server authentication, the client and the Group Manager leverage protocol-specific profiles of ACE. These include {{I-D.ietf-ace-dtls-authorize}} and {{I-D.ietf-ace-oscore-profile}}, as well as possible forthcoming profiles that comply with the requirements in Appendix C.3 of {{I-D.ietf-ace-oauth-authz}}.

## Terminology {#ssec-terminology}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}}{{RFC8174}} when, and only when, they appear in all capitals, as shown here.

Readers are expected to be familiar with the terms and concepts described in the ACE framework for authentication and authorization {{I-D.ietf-ace-oauth-authz}}. Message exchanges are presented as RESTful protocol interactions, for which HTTP {{RFC7231}} provides useful terminology.

The terminology for entities in the considered architecture is defined in OAuth 2.0 {{RFC6749}} and {{I-D.ietf-ace-actors}}. In particular, this includes Client (C), Resource Server (RS), and Authorization Server (AS). Terminology for constrained environments, such as "constrained device" and "constrained-node network", is defined in {{RFC7228}}.

Readers are expected to be familiar with the terms and concepts related to the CoAP protocol described in {{RFC7252}}{{RFC7390}}. Note that, unless otherwise indicated, the term "endpoint" is used here following its OAuth definition, aimed at denoting resources such as /token and /introspect at the AS and /authz-info at the RS. This document does not use the CoAP definition of "endpoint", which is "An entity participating in the CoAP protocol".

Readers are expected to be familiar with the terms and concepts for protection and processing of CoAP messages through OSCORE {{I-D.ietf-core-object-security}} also in group communication contexts {{I-D.ietf-core-oscore-groupcomm}}.

This document refers also to the following terminology.

* Joining node: a network node intending to join an OSCORE group, where communication is based on CoAP {{RFC7390}} and secured with OSCORE as described in {{I-D.ietf-core-oscore-groupcomm}}.

* Join process: the process through which a joining node becomes a member of an OSCORE group. The join process is enforced and assisted by the Group Manager responsible for that group.

* Join resource: a resource hosted by the Group Manager, associated to an OSCORE group under that Group Manager. A join resource is identifiable with the Group Identifier (Gid) of the respective group. A joining node accesses a join resource to start the join process and become a member of that group.

* Join endpoint: an endpoint at the Group Manager associated to a join resource.

# Protocol Overview {#sec-protocol-overview}

Group communication for CoAP over IP multicast has been enabled in {{RFC7390}} and can be secured with Object Security for Constrained RESTful Environments (OSCORE) {{I-D.ietf-core-object-security}} as described in {{I-D.ietf-core-oscore-groupcomm}}. A network node explicitly joins an OSCORE group, by interacting with the responsible Group Manager. Once registered in the group, the new node can securely exchange messages with other group members.

This specification describes how a network node joins an OSCORE group leveraging the ACE framework for authentication and authorization {{I-D.ietf-ace-oauth-authz}}. With reference to the ACE framework and the terminology defined in OAuth 2.0 {{RFC6749}}:

* The Group Manager acts as Resource Server (RS), and hosts one join resource for each OSCORE group it manages. Each join resource is exported by a distinct join endpoint. During the join process, the Group Manager provides joining nodes with the parameters and keying material for taking part to secure communications in the group.

* The joining node acts as Client (C), and requests to join an OSCORE group by accessing the related join endpoint at the Group Manager.

* The Authorization Server (AS) enables and enforces the authorized access of joining nodes to join endpoints at the Group Manager, and hence the access to the related OSCORE groups. Multiple Group Managers can be associated to the same AS. The AS is not necessarily expected to release Access Tokens for any other purpose than accessing join resources on registered Group Managers. However, the AS may be configured also to release Access Tokens for accessing resources at members of OSCORE groups.

All communications between the involved entities rely on the CoAP protocol and MUST be secured. The joining node and the Group Manager leverage protocol-specific profiles of ACE to achieve communication security, proof-of-possession and server authentication. To this end, the AS MUST signal the specific profile to use, consistently with requirements and assumptions defined in the ACE framework {{I-D.ietf-ace-oauth-authz}}.

Communications between the joining node and the AS (/token endpoint) as well as between the Group Manager and the AS (/introspection endpoint) can be secured by different means, for instance by means of DTLS {{RFC6347}} or OSCORE {{I-D.ietf-core-object-security}}. Further details on how the AS secures communications (with the joining node and the Group Manager) depend on the specifically used profile of ACE, and are out of the scope of this specification.

The following steps are performed for joining an OSCORE group. Exchanged messages follow the formats defined in [NEW_DRAFT], and are further specified in {{sec-joining-node-to-AS}} and {{sec-joining-node-to-GM}}. Besides, the Group Manager reflects the entity Key Distribution Center (KDC) referred in [NEW_DRAFT].

1. The joining node requests an Access Token from the AS to access a join resource on the Group Manager (see {{sec-joining-node-to-AS}}). The response from the AS enables the joining node to start a secure channel with the Group Manager, if not already established. The joining node can also contact the AS for updating a previously released Access Token, in order to access further groups under the same Group Manager (see {{sec-updating-authorization-information}}).

2. The joining node transfers authentication and authorization information to the Group Manager by posting the Access Token. Then, the joining node and the Group Manager have to establish a secure channel in case one is not already set up (see {{sec-joining-node-to-GM}}). That is, a joining node MUST establish a secure communication channel with a Group Manager, before joining an OSCORE group under that Group Manager for the first time.

3. The joining node starts the join process to become a member of the OSCORE group, by accessing the related join resource hosted by the Group Manager (see {{sec-joining-node-to-GM}}). A same Access Token can authorize the joining node to access multiple groups under the same Group Manager. In such a case, the joining node sequentially performs multiple join processes with the Group Manager, separately for each group to join and by accessing the respective join endpoint.

4. At the end of the join process, the joining node has received from the Group Manager the parameters and keying material to securely communicate in the OSCORE group.

# Joining Node to Authorization Server {#sec-joining-node-to-AS}

This section considers a joining node that intends to contact the Group Manager for the first time. That is, the joining node has never attempted before to join an OSCORE group under that Group Manager. Also, the joining node and the Group Manager do not have a secure communication channel established.

In case the specific AS associated to the Group Manager is unknown to the joining node, the latter can rely on mechanisms like the Unauthorized Resource Request message described in Section 2.1 of {{I-D.ietf-ace-dtls-authorize}} to discover the correct AS in charge of the Group Manager. As an alternative, the joining node may look up in a Resource Directory service {{I-D.ietf-core-resource-directory}}.

## Authorization Request {#ssec-auth-req}

The joining node contacts the AS, in order to request an Access Token for accessing the join resource(s) hosted by the Group Manager. The Access Token request sent to the /token endpoint follows the format of the Authorization Request message defined in Section 3.1 of [NEW_DRAFT]. In particular:

* The "aud" parameter is set to the address of the Group Manager.

* The "scope" parameter includes:

- in the first element, the identifier of the join resource at the Group Manager related to the group that the joining node intends to join. This identifier may not fully coincide with the Gid currently associated to the respective group, e.g. if it includes a dynamic component.

* in the second parameter, which MUST be present, the role(s) that the joining node intends to have in the group it intends to join. Roles and their combinations are defined in {{I-D.ietf-core-oscore-groupcomm}}, and indicated as "multicaster", "listener" and "purelistener". Multiple roles are specified in the form of a CBOR array.

* The "get_pub_keys" parameter is present only if the Group Manager is configured to store the public keys of the group members and, at the same time, the joining node wants to retrieve such public keys during the joining process (see {{sec-public-keys-of-joining-nodes}}). In any other case, this parameter MUST NOT be present.

## Authorization Response {#ssec-auth-resp}

The AS is responsible for authorizing the joining node, accordingly to group join policies enforced on behalf of the Group Manager. In case of successful authorization, the AS releases an Access Token bound to a proof-of-possession key associated to the joining node. The same Access Token can authorize the joining node to access multiple groups under the same Group Manager.

Then, the AS provides the joining node with the Access Token, together with an Access Token response which follows the format of the Authorization Response message defined in Section 3.2 of [NEW_DRAFT]. Also, the Access Token response indicates the specific profile of ACE to use for securing communications between the joining node and the Group Manager.

In particular, if symmetric keys are used, the AS generates a proof-of-possession key, binds it to the Access Token, and provides it to the joining node in the "cnf" parameter of the  Access Token response. Instead, if asymmetric keys are used, the joining node provides its own public key to the AS in the "cnf" parameter of the Access Token request. Then, the AS uses it as proof-of-possession key bound to the Access Token, and provides the joining node with the Group Manager's public key in the "rs_cnf" paramenter of the Access Token response.

# Joining Node to Group Manager {#sec-joining-node-to-GM}

First, the joining node posts the Access Token to the /authz-info endpoint at the Group Manager, in accordance with the Token post considered in Section 3.3 of [NEW_DRAFT]. Then, the joining node establishes a secure channel with the Group Manager, according to what specified in the Access Token response and to the signalled profile of ACE.

## Join Request {#ssec-join-req}

Once a secure communication channel with the Group Manager has been established, the joining node requests to join the OSCORE group of interest, by accessing the related join resource at the Group Manager.

In particular, the joining node sends to the Group Manager a confirmable CoAP request, using the method POST and targeting the join endpoint associated to that group. This join request follows the format of the Keying Distribution request defined in Section 4.1 of [NEW_DRAFT]. In particular:

<!--
* The "aud" parameter MUST be present and is set to the multicast IP address associated to the group.
-->

<!--
* The "scope" parameter MUST be present and is set to the Group Identifier (Gid) of the group, as known to the joining node at this point in time. This may not fully coincide with the Gid currently associated to the group, e.g. if it includes a dynamic component.

* The "role" parameter has the same value as in the Authorization Request previously sent to the AS.
-->

<!--
* The "client_id" parameter is present if included also in the Authorization Request previously sent to the AS. In such a case, its value is the same as in the Authorization Request. Otherwise, this parameter MUST NOT be present.-->

* The "get_pub_keys" parameter is present only if included also in the Authorization Request previously sent to the AS. In such a case, its value is the same as in the Authorization Request. Otherwise, this parameter MUST NOT be present.

* The "cnf" parameter is present only if public keys are used as proof-of-possession keys and, at the same time, the Group Manager is configured to store the public keys of the group members. In such a case, this parameter contains the public key of the joining node. Otherwise, this parameter MUST NOT be present.

* The "cert" parameter is present only if public keys are used as proof-of-possession keys and, at the same time, the Group Manager is not configured to store the public keys of the group members. In such a case, this parameter contains a public certificate of the joining node. Otherwise, this parameter MUST NOT be present.

* The "pub_keys_repos" parameter may be present if the "cert" parameter is present. In such a case, this parameter contains the list of public key repositories storing the certificate of the joining node. In case the "cert" parameter is not present, this parameter MUST NOT be present. 

## Join Response {#ssec-join-resp}

The Group Manager processes the request according to {{I-D.ietf-ace-oauth-authz}}. If this yields to a positive response, the Group Manager updates the group membership by registering the joining node as a new member of the group.

Then, the Group Manager replies to the joining node providing the information specified in Appendix D.1 of {{I-D.ietf-core-oscore-groupcomm}}. This join response follows the format of the Keying Distribution success response defined in Section 4.2 of [NEW_DRAFT]. In particular:

<!--
* The "role" indicates the effective role(s) that the joining node can take in the group. This parameter can be omitted if the Group Manager confirms all the same roles indicated in the join request.-->

* The "key" parameter includes what the joining node needs to set up the OSCORE Security Context (see Section 3 of {{I-D.ietf-core-oscore-groupcomm}}).

   * The "kty" parameter has value "Symmetric".

   * The "k" parameter includes the OSCORE Master Secret.

   * The "alg" parameter, if present, has as value the AEAD algorithm used in the group.

   * The "kid" parameter, if present, has as value the identifier of the key in the parameter "k".

   * The "base IV" parameter, if present, has as value the OSCORE Common IV.

   * The "clientID" parameter MUST be present and has as value the OSCORE Endpoint ID assigned to the joining node by the Group Manager.

   * The "serverID" parameter MUST be present and has as value the Group Identifier (Gid) currently associated to the group.

   * The "kdf" parameter, if present, has as value the KDF algorithm used in the group.

   * The "slt" parameter, if present, has as value the OSCORE Master Salt.

   * The "cs_alg" parameter MUST be present and has as value the countersignature algorithm used in the group.

* The "pub_keys" parameter is present only if the "get_pub_keys" parameter was present in the join request. This parameter includes the public keys of the group members.

* The "group_policies" parameter SHOULD be present and includes a list of key words indicating particular policies enforced in the group. For instance, it can indicate the method to achieve synchronization of sequence numbers among group members (see Appendix E of {{I-D.ietf-core-oscore-groupcomm}}), as well as the rekeying protocol used to renew the keying material in the group (see Section 3.1 of {{I-D.ietf-core-oscore-groupcomm}}).

* The "mgt_key_material" parameter SHOULD be present and includes the administrative keying material that the joining node requires to participate in the rekeying process led by the Group Manager. The exact content and format depends on the specific rekeying scheme used in the group.

Finally, the joining node uses the information received in the join response to set up the OSCORE Security Context, as described in Section 2 of {{I-D.ietf-core-oscore-groupcomm}}. From then on, the joining node can exchange group messages secured with OSCORE as described in Section 4 of {{I-D.ietf-core-oscore-groupcomm}}.

# Public Keys of Joining Nodes # {#sec-public-keys-of-joining-nodes}

Source authentication of OSCORE messages exchanged within the group is ensured by means of digital counter signatures {{I-D.ietf-core-oscore-groupcomm}}. Therefore, group members must be able to retrieve each other's public key from a trusted key repository, in order to verify the source authenticity of incoming group messages.

Upon joining an OSCORE group, a joining node is expected to make its own public key available to the other group members, either through the Group Manager or through another trusted, publicly available, key repository. However, this is not required for a node that joins a group exclusively as pure listener.

As also discussed in Section 6 of {{I-D.ietf-core-oscore-groupcomm}}, it is recommended that the Group Manager is configured to store the public keys of the group members and to provide them upon request. In so, three cases can occur.

* The Group Manager already acquired the public key of the joining node during a previous join process. In this case, the joining node is not required to provide again its own public key to the Group Manager.

* The joining node and the Group Manager use an asymmetric proof-of-possession key to establish a secure communication channel. In this case, the Group Manager stores the proof-of-possession key conveyed in the Access Token as the public key of the joining node.

* The joining node and the Group Manager use a symmetric proof-of-possession key to establish a secure communication channel. In this case, upon performing a join process with that Group Manager for the first time, the joining node specifies its own public key as "Identity credentials" of the join request targeting the join endpoint (see Appendix D.1 of {{I-D.ietf-core-oscore-groupcomm}}). In particular, the joining node includes its own public key in the "cnf" parameter of the join request.

Then, the Group Manager MUST verify that the joining node actually owns the associated private key, by performing a proof-of-possession challenge-response.

Furthermore, as described in {{ssec-join-req}}, the joining node may have explicitly requested the Group Manager to retrieve the public keys of the current group members, i.e. through the "get_pub_keys" parameter in the join request. In this case, the Group Manager includes also such public keys in the "cnf_pub_keys" parameter of the join response.

On the other hand, in case the Group Manager is not configured to store public keys of group members, the joining node provides the Group Manager with its own certificate in the "cert" parameter of the join request (see Appendix D.2 of {{I-D.ietf-core-oscore-groupcomm}}).

# Updating Authorization Information {#sec-updating-authorization-information}

At any point in time, a node might want to join further OSCORE groups under the same Group Manager. In such a case, the joining node requests from the AS an updated Access Token for accessing the new OSCORE groups of interest.

The joining node uploads the new Access Token to the /authz-info resource at the Group Manager, using the already established secure communication channel. After that, the joining node performs the joining process described in {{sec-joining-node-to-GM}}, separately for each OSCORE group to join.

Since the joining node and the Group Manager already share a secure communication channel, they are not required to establish a new one. However, according to the specific profile of ACE in use, the joining node and the Group Manager may leverage the new Access Token to establish a new secure communication channel or update the currently existing one.

# Security Considerations {#sec-security-considerations}

The method described in this document leverages the following management aspects related to OSCORE groups and discussed in the sections of {{I-D.ietf-core-oscore-groupcomm}} indicated below.

* Management of group keying material (Section 3.1). This includes the need to revoke and renew the keying material currently used in the OSCORE group, upon changes in the group membership. In particular, renewing the keying material is required upon a new node joining the group, in order to preserve backward security. The Group Manager is responsible to enforce rekeying policies and accordingly update the keying material within the groups of its competence.

* Synchronization of sequence numbers (Section 6). This concerns how a listener node that has just joined an OSCORE group can synchronize with the sequence number of multicasters in the same group.

* Provisioning and retrieval of public keys (Appendix C.2). This provides guidelines about how to ensure the availability of group members' public keys, possibly relying on the Group Manager as trusted key repository.

Further security considerations are inherited from the ACE framework for Authentication and Authorization {{I-D.ietf-ace-oauth-authz}}, as well as from the specific profile of ACE signalled by the AS, such as {{I-D.ietf-ace-dtls-authorize}} and {{I-D.ietf-ace-oscore-profile}}.

# IANA Considerations {#sec-iana}

This document has no actions for IANA.

# Acknowledgments {#sec-acknowledgments}

The authors sincerely thank Santiago Arag&oacute;n, Stefan Beck, Martin Gunnarsson, Francesca Palombini, Jim Schaad, Ludwig Seitz and G&ouml;ran Selander for their comments and feedback.

The work on this document has been partly supported by the EIT-Digital High Impact Initiative ACTIVE.

--- back
