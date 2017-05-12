---
title: IPsec Profile for ACE
abbrev: IPsec-ACE
docname: draft-aragon-ace-ipsec-profile
date: 2017-04-25
category: std

ipr: trust200902
area: Security
workgroup: ACE Working Group
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: S. Aragón
    name: Santiago Aragón
    organization: SEEMOO@TU Darmstadt
    street: Mornewegstr. 32 (S4/14)
    city: Darmstadt
    code: DE-64293
    country: Germany
    email: santiago9101@gmail.com

 -
    ins: M. Tiloca
    name: Marco Tiloca
    org: RISE SICS
    street: Box 1263
    city: Kista
    code: SE-164 29
    country: Stockholm
    phone: +46 70 604 65 01
    email: marco.tiloca@ri.se

 -
    ins: S. Raza
    name: Shahid Raza
    org: RISE SICS
    street: Box 1263
    city: Kista
    code: SE-164 29
    country: Stockholm
    phone: +46 76 883 17 97
    email: shahid@sics.se


normative:
  RFC2119:


informative:
  RFC5389:
  RFC7252:
  RFC4301:
  RFC7296:
  RFC4302:
  RFC4303:
  I-D.ietf-ace-oauth-authz:
  I-D.selander-ace-cose-ecdhe:



--- abstract

TODO
--- middle

# Introduction        {#problems}

TODO motivate why not oscoap not dtls?

This document defines a profile of the ACE framework {{I-D.ietf-ace-oauth-authz}}. In this profile, a client (C) and a resource server (RS) communicate using CoAP {{RFC7252}} over IPsec {{RFC4301}}. C uses an access token bound to a key (proof-of-possession key) to access RS. The successful establishment of a IPsec connection between C and RS provides communication confidentiality, proof-of-possession as well as RS and C mutual authentication. Moreover, this profiles provides the opportunity to guarantee the goals defined in Section 2.1 of {{RFC4301}}, e.g., Integrity and IP spoofing protection, limited traffic flow confidentiality, rejection of
replays, enables the usage of Virtual Private Networks (VPN) and leverages the implementation of built-in link-layer encryption hardware.

The CoAP-Ipsec profile for ACE extends the flexibility of the Ipsec framework where the selection of security protocols, i.e. Encapsulating Security Payload (ESP) and/or IP Authentication Header (AH), key management, and modes of operations, i.e., tunnel or transport. Those parameters are defined by a Security Association (SA) and yield to different security guarantees to fulfill distinct network settings requirements.

Moreover, this specification supports three key establishment techniques: via pre-establishment of an SA pair; using Ephemeral Diffie-Hellman Over COSE (EDHOC) key exchange{{I-D.selander-ace-cose-ecdhe}}; or employing the Internet Key Exchange Protocol version 2 (IKEv2) {{RFC7296}}. The output of the aforementioned techniques provide the corresponding randomness and additional parameters to set up an IP connection between C and RS. Such parameters are assumed to be previously negotiated between RS and the Authentication Server (AS).

Add something about the proposed settings





## Terminology           {#Terminology}
In this document, the key words "MUST", "MUST NOT", "REQUIRED",
"SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY",
and "OPTIONAL" are to be interpreted as described in BCP 14, RFC 2119
{{RFC2119}} and indicate requirement levels for compliant STuPiD
implementations.



# Protocol Definition   {#prot}

In this section, we define how C request a resource from RS though an access token issued by AS as specified in {{I-D.ietf-ace-oauth-authz}}. In particular, it defines the transfer of the necessary information to start an IPsec channel between C and RS, i.e., the agreement of a pair of SAs. For simplicity, we divide the protocol into three parts showed in figure {{figprot}} and detailed in the rest of this section.

~~~~~~~~~~
            C                            RS                   AS
            |                            |                     |
            | [-- Resource Request --->] |                     |
    (1)     |                            |                     |
            | [<----- AS Information --] |                     |
            |                            |                     |
    ---     |                            |                     |
            |                            |                     |
            | --- Token Request  ----------------------------> |
    (2)     |                            |                     |
            | <----------- Access Token + RS Information------ |
            |                            |                     |
    ---     |                            |                     |
            |                            |                     |
            | ----- Access Token ------> |                     |
            |                            |                     |
    (3)     | [<===== SA setup =======>] |                     |
            |                            |                     |
            | ==== Resource Request ===> |                     |
            |                            |                     |
            | <=== Resource Response === |                     |
            |                            |                     |
~~~~~~~~~~
{: #figprot title="Protocol Overview"}


## Unauthorized Client-to-RS {#prot1}
The first protocol phase (See (1) in {{figprot}}) aims to provide C with enough information to reach the AS. It may include an optional unauthorized request to the RS where C has not been granted with the corresponding authorization. This request helps C to determine the AS where the Token Request should be addressed. The request is denied by the RS and a response containing AS information is sent to C.

## Client-to-AS {#prot2}

The phase (See (2) in {{figprot}}) consists in the request of an Access Token to grant authenticated access to a resource and secure the communication between C and RS. C submits an Access Token Request to the /token endpoint at AS as specified in Section 5.5.1 of {{I-D.ietf-ace-oauth-authz}}. Figure 2 and Figure 3 of {{I-D.ietf-ace-oauth-authz}} provide examples of the aforementioned request.

If AS was able to verify the validity of the access token request and the client was authorized to access the resource as described in his request, AS responds with a CoAP message with code 2.01 as specified in Section 5.5.2 of {{I-D.ietf-ace-oauth-authz}}. AS can specify that the use of the CoAP-Ipsec profile is required to communicate with RS. To signalize the use of this profile is required that the AS assigns the value of "coap_ipsec" to the "profile" field in the Access Token. Error responses are handled as described in Section 5.5.3 of {{I-D.ietf-ace-oauth-authz}}.

The communication between AS and C should be done in a secure fashion, e.g. in section 5.5 of {{I-D.ietf-ace-oauth-authz}} it is specified how to use OSCOAP with pre-established credentials to secure the interactions with the /token endpoint. However, other security protocols, e.g. IPSec or DTLS may be used.


### Key Management for Ipsec {#SA_est}

In this section we detail how the establishment of the necessary key material and protocol parameters to set up the necessary Security Associations (SA) between C and RS and delegate the authentication and authorization to a successfully established Ipsec connection. Since an SA represent a simplex connection between two parties, for a duplex connection 2 SA are required.
Every SA definition must include a Security Parameter Index (SPI), lifetime, IPsec protocol mode, security protocol, authentication algorithm, encryption algorithm and key.

SA
EDHOC
<!--
  EDHOC specifies an
 authenticated Diffie-Hellman protocol that allows two parties to use
 CBOR [RFC7049] and COSE in order to establish a shared secret key
 with perfect forward secrecy.

 Alternatively the pop-key (symmetric or asymmetric) MAY be used to
 authenticate the messages in the key exchange protocol EDHOC {{I-D.selander-ace-cose-ecdhe}}, from which a master secret is derived.

If EDHOC is used together with OSCOAP, and the pop-key (symmetric or
 asymmetric) is used to authenticate the messages in EDHOC, then the
 AS MUST provision the following data, in response to the access token
 request:
 o a symmetric or asymmetric key (pop-key)
 o if the pop-key is symmetric, a key identifier;
 How these parameters are communicated depends on the type of key
 (asymmetric or symmetric).

+ PreSharedKey
 In the case of a symmetric key, the AS MUST communicate the key to
 the client in the ’cnf’ parameter of the access token response, as
 specified in section 5.5.2. of [I-D.ietf-ace-oauth-authz]. AS MUST
 also select a key identifier, that MUST be included as the ’kid’
 parameter either directly in the ’cnf’ structure, as in figure 4 of
 [I-D.ietf-ace-oauth-authz], or as the ’kid’ parameter of the
 COSE_key, as in figure 3 of [I-D.ietf-ace-oauth-authz].
+ RPK
To provide forward secrecy and mutual authentication in the case of
 pre-shared keys, pre-established raw public keys or with X.509
 certificates it is RECOMMENDED to use EDHOC I-D.selander-ace-cose-
 ecdhe] to generate the keying material. EDHOC MUST be used as
 defined in Appendix C, with the following additions and
 modifications.
IKEV2
+ PreSharedKey
+ RPK
-->
TODO define how to specify the km method

#### Direct SA establishment {#dirSA}
asd

#### EDHOC {#edhoc}
as

#### IKEv2 {#ike}
asd

## Client-to-RS {#prot3}

During the part of the protocol showed in (3) of {{figprot}} C submits the corresponding Access Token to the \authz-info endpoint at RS. The details of this request, as well as the handling of invalid tokens by RS, is defined in Section 5.7.1 of {{I-D.ietf-ace-oauth-authz}}. The Access Token should specify one of the SA establishments defined in {{SA_est}}. If Direct SA establishment is specified, C and RS skip the optional SA setup showed in {{figprot}} since the Access Token response, contains a pair of SAs that allows C and RS to directly start a IPsec channel as specified in the SA pair. Otherwise, EDHOC (resp. IKEv2) key management should have been specified as defined in {{edhoc}} (resp. {{ike}}).

Independently of the chosen SA setup mechanism, if after establishing successfully an IPsec channel, C holds an valid Access Token but it does not authorize the access to the requested resource, RS must send a 4.03 (Forbidden) response. Similarly, if the Token does not cover the intended action, RS must send a 4.05 (Method Not Allowed) response.

### SA setup via EDHOC {#edhoc_setup}

To derive a SA pair from an EDHOC key establishment the parties C and RS uses EDHOC to derive the only missing information to complete a SA pair, i.e., the encryption and authentication keys for the corresponding security protocols signalized by the AS.

C also includes message_1 of the EDHOC protocol with asymmetric (resp. symmetric) keys as described in section 4.2 (resp. section 5.2) of {{I-D.selander-ace-cose-ecdhe}}. In case that asymmetric keys are used, RS communicates its public key to the client using the Access Token Response as stated in section 5.5.2 of {{I-D.ietf-ace-oauth-authz}}.

If the validity of the Access Token is verified then RS continues the EDHOC exchange as described in Appendix C.1 of {{I-D.selander-ace-cose-ecdhe}}, otherwise it responds with the corresponding error code. Otherwise, if any step of EDHOC verification fails, RS must send a 4.01 (Unauthorized request) response.

If no error response was produce, C and RS are able to perform any further communication using IPsec as defined in the SA pair. We stress that it is expected that if the IPsec profile is specified by AS, /authz-info endpoint must be able to process all and handle of the EDHOC message exchange and SA configuration as defined before.

### SA setup via IKEv2 {#ike_setup}

If the use of IKEv2 is signalized to establish a SA pair, a symmetric, i.e. shared secret, or asymmetric, i.e., public key signatures, cryptography is used to authenticate IKE messages as described in section 2.15 of {{RFC7296}}. In the case of the former authentication method, a shared secret must be included in the Access Token Response and contained or referenced with the corresponding ID in the Access Token. If the latter mechanism is set, the public key of RS is communicated to C using the Access Token Response. The protocol should be executed between C and RS as described in section 2 of {{RFC7296}} with no further modifications.


# Security Considerations {#sec_con}
TODO

# IANA Considerations {#iana_con}
TODO

# Acknowledgments {#acknow}
TODO EIT

--- back



Examples  {#xmp}
========

This appendix provides some examples of the STuPiD protocol operation.

~~~~~~~~~~
   Request:

      GET /stupid.php HTTP/1.0
      User-Agent: Example/1.11.4
      Accept: */*
      Host: example.org
      Connection: Keep-Alive

   Response:

      HTTP/1.1 200 OK
      Date: Sun, 05 Jul 2009 00:30:37 GMT
      Server: Apache/2.2
      Cache-Control: no-cache, must-revalidate
      Expires: Sat, 26 Jul 1997 05:00:00 GMT
      Vary: Accept-Encoding
      Content-Length: 17
      Keep-Alive: timeout=1, max=400
      Connection: Keep-Alive
      Content-Type: application/octet-stream

      192.0.2.239:36654
~~~~~~~~~~
{: #figxmpdisco title="Discovering External IP Address and Port"}