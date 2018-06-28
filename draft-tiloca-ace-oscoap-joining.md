---
title: Joining OSCORE groups in ACE
abbrev: OSCORE group joining in ACE
docname: draft-tiloca-ace-oscoap-joining-04
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
    org: RISE SICS
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
  I-D.palombini-ace-key-groupcomm:
  I-D.ietf-ace-oauth-authz:
  I-D.ietf-ace-oscore-profile:

informative:
  I-D.ietf-ace-dtls-authorize:
  RFC6347:
  RFC6749:
  RFC7390:
  RFC8152:

--- abstract

This document describes a method to join a group where communications are based on CoAP and secured with Object Security for Constrained RESTful Environments (OSCORE). The proposed method delegates the authentication and authorization of client nodes that join an OSCORE group through a Group Manager server. This approach builds on the ACE framework for Authentication and Authorization, and leverages protocol-specific profiles of ACE to achieve communication security, proof-of-possession and server authentication.

--- middle

# Introduction {#sec-introduction}

Object Security for Constrained RESTful Environments (OSCORE) {{I-D.ietf-core-object-security}} is a method for application-layer protection of the Constrained Application Protocol (CoAP) {{RFC7252}}, using CBOR Object Signing and Encryption (COSE) {{RFC8152}} and enabling end-to-end security of CoAP payload and options.

As described in {{I-D.ietf-core-oscore-groupcomm}}, OSCORE may be used also to protect CoAP group communication over IP multicast {{RFC7390}}. This relies on a Group Manager entity, which is responsible for managing an OSCORE group, where members exchange CoAP messages secured with OSCORE. In particular, the Group Manager coordinates the join process of new group members and can be responsible for multiple groups.

This specification builds on the ACE framework for Authentication and Authorization {{I-D.ietf-ace-oauth-authz}} and defines how a client joins an OSCORE group through a resource server acting as Group Manager. The client acting as joining node relies on an Access Token, which is bound to a proof-of-possession key and authorizes the access to a specific join resource at the Group Manager. Messages exchanged among the participants follow the formats defined in {{I-D.palombini-ace-key-groupcomm}} for provisioning keying material in group communication scenarios.

In order to achieve communication security, proof-of-possession and server authentication, the client and the Group Manager leverage protocol-specific profiles of ACE. These include also possible forthcoming profiles that comply with the requirements in Appendix C of {{I-D.ietf-ace-oauth-authz}}.

## Terminology {#ssec-terminology}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}}{{RFC8174}} when, and only when, they appear in all capitals, as shown here.

Readers are expected to be familiar with the terms and concepts described in the ACE framework for authentication and authorization {{I-D.ietf-ace-oauth-authz}}. The terminology for entities in the considered architecture is defined in OAuth 2.0 {{RFC6749}}. In particular, this includes Client (C), Resource Server (RS), and Authorization Server (AS).

Readers are expected to be familiar with the terms and concepts related to the CoAP protocol described in {{RFC7252}}{{RFC7390}}. Note that, unless otherwise indicated, the term "endpoint" is used here following its OAuth definition, aimed at denoting resources such as /token and /introspect at the AS and /authz-info at the RS. This document does not use the CoAP definition of "endpoint", which is "An entity participating in the CoAP protocol".

Readers are expected to be familiar with the terms and concepts for protection and processing of CoAP messages through OSCORE {{I-D.ietf-core-object-security}} also in group communication scenarios {{I-D.ietf-core-oscore-groupcomm}}. These include the concept of Group Manager, as the entity reponsible for a set of groups where communications are secured with OSCORE. In this specification, the Group Manager acts as Resource Server.

This document refers also to the following terminology.

* Joining node: a network node intending to join an OSCORE group, where communication is based on CoAP {{RFC7390}} and secured with OSCORE as described in {{I-D.ietf-core-oscore-groupcomm}}.

* Join process: the process through which a joining node becomes a member of an OSCORE group. The join process is enforced and assisted by the Group Manager responsible for that group.

* Join resource: a resource hosted by the Group Manager, associated to an OSCORE group under that Group Manager. A join resource is identifiable with the Group Identifier (Gid) of the respective group. A joining node accesses a join resource to start the join process and become a member of that group.

* Join endpoint: an endpoint at the Group Manager associated to a join resource.

* Requester: member of an OSCORE group that sends request messages to other members of the group.

* Listener: member of an OSCORE group that receives request messages from other members of the group. A listener may reply back, by sending a response message to the requester which has sent the request message.

* Pure listener: member of a group that is configured as listener and never replies back to requesters after receiving request messages. This corresponds to the term "silent server" used in {{I-D.ietf-core-oscore-groupcomm}}.

# Protocol Overview {#sec-protocol-overview}

Group communication for CoAP over IP multicast has been enabled in {{RFC7390}} and can be secured with Object Security for Constrained RESTful Environments (OSCORE) {{I-D.ietf-core-object-security}} as described in {{I-D.ietf-core-oscore-groupcomm}}. A network node explicitly joins an OSCORE group, by interacting with the responsible Group Manager. Once registered in the group, the new node can securely exchange messages with other group members.

This specification describes how a network node joins an OSCORE group by using the ACE framework for authentication and authorization {{I-D.ietf-ace-oauth-authz}}. With reference to the ACE framework and the terminology defined in OAuth 2.0 {{RFC6749}}:

* The Group Manager acts as Resource Server (RS), and hosts one join resource for each OSCORE group it manages. Each join resource is exported by a distinct join endpoint. During the join process, the Group Manager provides joining nodes with the parameters and keying material for taking part to secure communications in the group.

* The joining node acts as Client (C), and requests to join an OSCORE group by accessing the related join endpoint at the Group Manager.

* The Authorization Server (AS) authorizes joining nodes to join OSCORE groups under their respective Group Manager. Multiple Group Managers can be associated to the same AS. The AS MAY release Access Tokens for other purposes than joining OSCORE groups under registered Group Managers. For example, the AS may also release Access Tokens for accessing resources hosted by members of OSCORE groups.

All communications between the involved entities rely on the CoAP protocol and must be secured.

In particular, communications between the joining node and the Group Manager leverage protocol-specific profiles of ACE to achieve communication security, proof-of-possession and server authentication. To this end, the AS must signal the specific profile to use, consistently with requirements and assumptions defined in the ACE framework {{I-D.ietf-ace-oauth-authz}}.

With reference to the AS, communications between the joining node and the AS (/token endpoint) as well as between the Group Manager and the AS (/introspect endpoint) can be secured by different means, for instance by means of DTLS {{RFC6347}} or OSCORE {{I-D.ietf-core-object-security}}. Further details on how the AS secures communications (with the joining node and the Group Manager) depend on the specifically used profile of ACE, and are out of the scope of this specification.

The following steps are performed for joining an OSCORE group. Messages exchanged among the participants follow the formats defined in {{I-D.palombini-ace-key-groupcomm}}, and are further specified in {{sec-joining-node-to-AS}} and {{sec-joining-node-to-GM}} of this document. The Group Manager acts as the Key Distribution Center (KDC) referred in {{I-D.palombini-ace-key-groupcomm}}.

1. The joining node requests an Access Token from the AS, in order to access a join resource on the Group Manager and hence join the associated OSCORE group (see {{sec-joining-node-to-AS}}). The joining node will start or continue using a secure communication channel with the Group Manager, according to the response from the AS.

2. The joining node transfers authentication and authorization information to the Group Manager by posting the obtained Access Token (see {{sec-joining-node-to-GM}}). After that, a joining node must have a secure communication channel established with the Group Manager, before starting to join an OSCORE group under that Group Manager (see {{sec-joining-node-to-GM}}). Possible alternatives to provide a secure communication channel include DTLS {{RFC6347}} and OSCORE {{I-D.ietf-core-object-security}}.

3. The joining node starts the join process to become a member of the OSCORE group, by accessing the related join resource hosted by the Group Manager (see {{sec-joining-node-to-GM}}).

4. At the end of the join process, the joining node has received from the Group Manager the parameters and keying material to securely communicate with the other members of the OSCORE group.

5. The joining node and the Group Manager maintain the secure channel, to support possible future communications.

# Joining Node to Authorization Server {#sec-joining-node-to-AS}

This section describes how the joining node interacts with the AS in order to be authorized to join an OSCORE group under a given Group Manager. In particular, it considers a joining node that intends to contact that Group Manager for the first time.

The message exchange between the joining node and the AS consists of the messages Authorization Request and Authorization Response defined in {{I-D.palombini-ace-key-groupcomm}}.

In case the specific AS associated to the Group Manager is unknown to the joining node, the latter can rely on mechanisms like the Unauthorized Resource Request message described in Section 2 of {{I-D.ietf-ace-dtls-authorize}} to discover the correct AS to contact.

## Authorization Request {#ssec-auth-req}

The joining node contacts the AS, in order to request an Access Token for accessing the join resource hosted by the Group Manager and associated to the OSCORE group. The Access Token request sent to the /token endpoint follows the format of the Authorization Request message defined in Section 3.1 of {{I-D.palombini-ace-key-groupcomm}}. In particular:

* The 'scope' parameter MUST be present and includes:

    - in the first element, the Group Identifier (Gid) of the group to join under the Group Manager. The value of this identifier may not fully coincide with the Gid value currently associated to the group, e.g. if the Gid is composed of a variable part such as a Group Epoch (see Appendix C of {{I-D.ietf-core-oscore-groupcomm}}).

    * in the second element, which MUST be present, the role(s) that the joining node intends to have in the group it intends to join. Possible values are: "requester"; "listener"; and "pure listener". Possible combinations are: "requester and listener"; and "requester and pure listener". Multiple roles are specified in the form of a CBOR array.

* The 'aud' parameter MUST be present and is set to the identifier of the Group Manager.

## Authorization Response {#ssec-auth-resp}

The AS is responsible for authorizing the joining node to join specific OSCORE groups, according to join policies enforced on behalf of the respective Group Manager.

In case of successful authorization, the AS releases an Access Token bound to a proof-of-possession key associated to the joining node.

Then, the AS provides the joining node with the Access Token as part of an Access Token response, which follows the format of the Authorization Response message defined in Section 3.2 of {{I-D.palombini-ace-key-groupcomm}}.

The 'exp' parameter MUST be present. Other means for the AS to specify the lifetime of Access Tokens are out of the scope of this specification.

The AS must include the 'scope' parameter in the response when the value included in the Access Token differs from the one specified by the joining node in the request. In such a case, the second element of 'scope' MUST be present and includes the role(s) that the joining node is actually authorized to take in the group, encoded as specified in {{ssec-auth-req}} of this document.

Also, the 'profile' parameter indicates the specific profile of ACE to use for securing communications between the joining node and the Group Manager (see Section 5.6.4.4 of {{I-D.ietf-ace-oauth-authz}}).

In particular, if symmetric keys are used, the AS generates a proof-of-possession key, binds it to the Access Token, and provides it to the joining node in the 'cnf' parameter of the  Access Token response. Instead, if asymmetric keys are used, the joining node provides its own public key to the AS in the 'cnf' parameter of the Access Token request. Then, the AS uses it as proof-of-possession key bound to the Access Token, and provides the joining node with the Group Manager's public key in the 'rs_cnf' parameter of the Access Token response.

# Joining Node to Group Manager {#sec-joining-node-to-GM}

First, the joining node posts the Access Token to the /authz-info endpoint at the Group Manager, in accordance with the Token post defined in Section 3.3 of {{I-D.palombini-ace-key-groupcomm}}. Then, the joining node establishes a secure channel with the Group Manager, according to what is specified in the Access Token response and to the signalled profile of ACE.

## Join Request {#ssec-join-req}

Once a secure communication channel with the Group Manager has been established, the joining node requests to join the OSCORE group, by accessing the related join resource at the Group Manager.

In particular, the joining node sends to the Group Manager a confirmable CoAP request, using the method POST and targeting the join endpoint associated to that group. This join request follows the format of the Key Distribution Request message defined in Section 4.1 of {{I-D.palombini-ace-key-groupcomm}}. In particular:

* The 'get_pub_keys' parameter is present only if the Group Manager is configured to store the public keys of the group members and, at the same time, the joining node wants to retrieve such public keys during the joining process (see {{sec-public-keys-of-joining-nodes}}). In any other case, this parameter MUST NOT be present.

* The 'client_cred' parameter, if present, includes the public key or certificate of the joining node. Specifically, it includes the public key of the joining node if the Group Manager is configured to store the public keys of the group members, or the certificate of the joining node otherwise. This parameter MAY be omitted if: i) public keys are used as proof-of-possession keys between the joining node and the Group Manager; or ii) the joining node is asking to access the group exclusively as pure listener; or iii) the Group Manager already acquired this information during a previous join process. In any other case, this parameter MUST be present.

* The 'pub_keys_repos' parameter MAY be present if the 'client_cred' parameter is both present and with value a certificate of the joining node. If present, this parameter contains the list of public key repositories storing the certificate of the joining node. In any other case, this parameter MUST NOT be present. 

## Join Response {#ssec-join-resp}

The Group Manager processes the request according to {{I-D.ietf-ace-oauth-authz}}. If this yields a positive outcome, the Group Manager updates the group membership by registering the joining node as a new member of the OSCORE group.

Then, the Group Manager replies to the joining node providing the information necessary to participate in the group communication. This join response follows the format of the Key Distribution success Response message defined in Section 4.2 of {{I-D.palombini-ace-key-groupcomm}}. In particular:

* The 'key' parameter includes what the joining node needs in order to set up the OSCORE Security Context as per Section 2 of {{I-D.ietf-core-oscore-groupcomm}}. In particular:

   * The 'kty' parameter has value "Symmetric".

   * The 'k' parameter includes the OSCORE Master Secret.

   * The 'exp' parameter specifies when the OSCORE Master Secret expires. 
   
   * The 'alg' parameter, if present, has as value the AEAD algorithm used in the group.

   * The 'kid' parameter, if present, has as value the identifier of the key in the parameter 'k'.

   * The 'base IV' parameter, if present, has as value the OSCORE Common IV.

   * The 'clientID' parameter, if present, has as value the OSCORE Endpoint ID assigned to the joining node by the Group Manager. This parameter is not present if the node joins the group exclusively as pure listener, according to what specified in the Access Token (see {{ssec-auth-resp}}). In any other case, this parameter MUST be present.

   * The 'serverID' parameter MUST be present and has as value the Group Identifier (Gid) currently associated to the group.

   * The 'kdf' parameter, if present, has as value the KDF algorithm used in the group.

   * The 'slt' parameter, if present, has as value the OSCORE Master Salt.

   * The 'cs_alg' parameter MUST be present and has as value the countersignature algorithm used in the group.

* The 'pub_keys' parameter is present only if the 'get_pub_keys' parameter was present in the join request. If present, this parameter includes the public keys of the group members that are relevant to the joining node. That is, it includes: i) the public keys of the non-pure listeners currently in the group, in case the joining node is configured (also) as requester; and ii) the public keys of the requesters currently in the group, in case the joining node is configured (also) as listener or pure listener.

* The 'group_policies' parameter SHOULD be present and includes a list of parameters indicating particular policies enforced in the group. For instance, it can indicate the method to achieve synchronization of sequence numbers among group members (see Appendix E of {{I-D.ietf-core-oscore-groupcomm}}), as well as the rekeying protocol used to renew the keying material in the group (see Section 2.1 of {{I-D.ietf-core-oscore-groupcomm}}).

* The 'mgt_key_material' parameter SHOULD be present and includes the administrative keying material that the joining node requires to participate in the rekeying process led by the Group Manager. The exact content and format depend on the specific rekeying protocol used in the group.

Finally, the joining node uses the information received in the join response to set up the OSCORE Security Context, as described in Section 2 of {{I-D.ietf-core-oscore-groupcomm}}. From then on, the joining node can exchange group messages secured with OSCORE as described in Section 4 of {{I-D.ietf-core-oscore-groupcomm}}.

When the OSCORE Master Secret expires, as specified by 'exp' in the 'key' parameter of the join response, the node considers the OSCORE Security Context also invalid and to be renewed. A possible approach for the node to renew the OSCORE Security Context through the Group Manager is described in Section 6 of {{I-D.palombini-ace-key-groupcomm}}. 

# Public Keys of Joining Nodes # {#sec-public-keys-of-joining-nodes}

Source authentication of OSCORE messages exchanged within the group is ensured by means of digital counter signatures {{I-D.ietf-core-oscore-groupcomm}}. Therefore, group members must be able to retrieve each other's public key from a trusted key repository, in order to verify the source authenticity of incoming group messages.

Upon joining an OSCORE group, a joining node is expected to make its own public key available to the other group members, either through the Group Manager or through another trusted, publicly available, key repository. However, this is not required for a node that joins a group exclusively as pure listener.

As also discussed in Section 6 of {{I-D.ietf-core-oscore-groupcomm}}, it is recommended that the Group Manager is configured to store the public keys of the group members and to provide them upon request. If so, three cases can occur when a new node joins a group.

* The Group Manager already acquired the public key of the joining node during a previous join process. In this case, the joining node may not provide again its own public key to the Group Manager, in order to limit the size of the join request.

* The joining node and the Group Manager use an asymmetric proof-of-possession key to establish a secure communication channel. In this case, the Group Manager stores the proof-of-possession key conveyed in the Access Token as the public key of the joining node.

* The joining node and the Group Manager use a symmetric proof-of-possession key to establish a secure communication channel. In this case, upon performing a join process with that Group Manager for the first time, the joining node specifies its own public key in the 'client_cred' parameter of the join request targeting the join endpoint (see {{ssec-join-req}}).

Furthermore, as described in {{ssec-join-req}}, the joining node may have explicitly requested the Group Manager to retrieve the public keys of the current group members, i.e. through the 'get_pub_keys' parameter in the join request. In this case, the Group Manager includes also such public keys in the 'pub_keys' parameter of the join response (see {{ssec-join-resp}}).

Later on as a group member, the node may need to retrieve the public keys of other group members. A possible approach to do this through the Group Manager is described in Section 7 of {{I-D.palombini-ace-key-groupcomm}}.

On the other hand, in case the Group Manager is not configured to store public keys of group members, the joining node provides the Group Manager with its own certificate in the 'client_cred' parameter of the join request targeting the join endpoint (see {{ssec-join-req}}). Then, the Group Manager validates and handles the certificate, for instance as described in Appendix D.2 of {{I-D.ietf-core-oscore-groupcomm}}.

# Security Considerations {#sec-security-considerations}

The method described in this document leverages the following management aspects related to OSCORE groups and discussed in the sections of {{I-D.ietf-core-oscore-groupcomm}} referred below.

* Management of group keying material (see Section 2.1 of {{I-D.ietf-core-oscore-groupcomm}}). This includes the need to revoke and renew the keying material currently used in the OSCORE group, upon changes in the group membership. In particular, renewing the keying material is required upon a new node joining the group, in order to preserve backward security. That is, the Group Manager should renew the keying material before completing the join process and sending a join response. Such a join response provides the joining node with the updated keying material just established in the group. The Group Manager is responsible to enforce rekeying policies and accordingly update the keying material in the groups of its competence (see Section 6 of {{I-D.ietf-core-oscore-groupcomm}}).

* Synchronization of sequence numbers (see Section 5 of {{I-D.ietf-core-oscore-groupcomm}}). This concerns how a listener node that has just joined an OSCORE group can synchronize with the sequence number of requesters in the same group.

* Provisioning and retrieval of public keys (see Appendix D.2 of {{I-D.ietf-core-oscore-groupcomm}}). This provides guidelines about how to ensure the availability of group members' public keys, possibly relying on the Group Manager as trusted key repository (see Section 6 of {{I-D.ietf-core-oscore-groupcomm}}).

Before sending the join response, the Group Manager should verify that the joining node actually owns the associated private key, for instance by performing a proof-of-possession challenge-response, whose details are out of the scope of this specification.

Further security considerations are inherited from the ACE framework for Authentication and Authorization {{I-D.ietf-ace-oauth-authz}}, as well as from the specific profile of ACE signalled by the AS, such as {{I-D.ietf-ace-dtls-authorize}} and {{I-D.ietf-ace-oscore-profile}}.

# IANA Considerations {#sec-iana}

This document has no actions for IANA.

# Acknowledgments {#sec-acknowledgments}

The authors sincerely thank Santiago Arag&oacute;n, Stefan Beck, Martin Gunnarsson, Francesca Palombini, Jim Schaad, Ludwig Seitz, G&ouml;ran Selander and Peter van der Stok for their comments and feedback.

The work on this document has been partly supported by the EIT-Digital High Impact Initiative ACTIVE.

--- back
