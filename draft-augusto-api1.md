---

stand_alone: yes
pi:
  rfcedstyle: yes
  toc: yes
  tocindent: yes
  tocdepth: 4
  sortrefs: yes
  symrefs: yes
  strict: yes
  comments: yes
  inline: yes
  text-list-symbols: o-\*+
  compact: yes
  subcompact: no

title: Secure Asset Transfer Protocol (SATP) API1 Specification
abbrev: SATP API1
docname: draft-augusto-api1-latest
category: info

ipr: trust200902
area: "Applications and Real-Time"
workgroup: "Secure Asset Transfer Protocol"

stream: IETF
keyword: Internet-Draft
consensus: true

venue:
  group: "Secure Asset Transfer Protocol"
  type: "Working Group"
  mail: "sat@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/sat/"
  github: "ietf-satp/draft-augusto-api1"
  latest: "https://ietf-satp.github.io/draft-augusto-api1/draft-augusto-api1.html"

author:

- ins: A. Augusto
  name: Andre Augusto
  organization: INESC-ID and Técnico Lisboa
  email: andre.augusto@tecnico.ulisboa.pt
- ins: R. Belchior
  name: Rafael Belchior
  organization: INESC-ID and Técnico Lisboa, Blockdaemon
  email: rafael.belchior@tecnico.ulisboa.pt
- ins: V. Ramakrishna
  name: Venkatraman Ramakrishna
  organization: IBM Research
  email: vramakr2@in.ibm.com
- ins: T. Hardjono
  name: Thomas Hardjono
  organization: MIT
  email: hardjono@mit.edu

normative:
  TLS: RFC8446
  HTTP: RFC2616

--- abstract

This memo describes the Gateway Client API (API1) for the Secure Asset Transfer (SAT) Protocol. The goal of this draft is to specify the schema of the interaction between a client application and a gateway that supports the SAT Protocol. This specification assures an implementation-agnostic interface for clients to interact with gateways, to manage transfers of digital assets across networks.

--- middle

# Introduction
{: #introduction}

The Secure Asset Transfer Protocol (SATP) is a gateway-to-gateway protocol that aims to standardize the communication between different networks to transfer digital assets. The transfer of assets is facilitated by a pair of gateways that support SATP. This document focuses on specifying the Gateway Client API (API1), which allows client applications to interact seamlessly with SATP-compliant gateways. This interaction enables secure and efficient asset transfers across various networks, interaction with third-party systems, and the management of SATP sessions.

API1 provides a standardized interface for client applications to communicate with their local SATP gateway. This interface ensures that clients can interact with gateways consistently, regardless of the underlying implementation of each one.

All messages exchanged between gateways and client applications are assumed to run over TLS1.2, and the endpoints at the respective gateways are associated with a certificate indicating the legal owner (or operator) of the gateway.

# Terminology
{: #terminology}

The following are some terminology used in the current document:

* Gateway: The computer system functionally capable of acting as a gateway in an asset transfer, that supports the Secure Asset Transfer Protocol.
* Client: The client application a user employs to interact with a gateway.

# Overview
{: #overview}

This document focuses on the specification of API1. Through API1, the Client transmits the necessary information to initiate SATP sessions or retrieve details about current or past sessions. This document outlines the API endpoints various message formats and flows to ensure that a Client can effectively utilize all functionalities provided by the Gateway.

The Client interacts with the Gateway either to retrieve information about its internal state or to trigger new digital asset transfers. The internal state of a Gateway includes data pertaining to previous digital asset transfers, logs related to previous actions (e.g., burning or minting assets), and proofs related to the current state of previously handled digital assets. Finally, the Gateway can also help the Client in querying external services discovering otherwise unavailable resources, or facilitating the retrieval of the supported networks to which gateways are connected.

```
                  +---------+
                  |  Client |
                  |  (App)  |
                  +---------+
                      |
                      |
                      V
                  +---------+
                  |   API1  |
    +---------+   +---------+------+   +------+---------+
    |         |   |         |      |   |      |         |
    | Network |---| Gateway | API2 |---| API2 | Gateway |
    |         |   |         |      |   |      |         |
    +---------+   +---------+------+   +------+---------+
                  |   API3  |
                  +---------+
                      |
                      |
                  +---------+
                  |         |
                  | External|
                  | System  |
                  +---------+
```
# API1 Message Flows
{: #api1-message-flows}

In this section, we specify all messages and respective payloads sent between the Client and the Gateway. This draft also leverages the definition of Message Formats and Payloads of the core SATP draft {{?I-D.draft-ietf-satp-core}}.

In this draft, similarly to the SATP core specification, Gateways MUST support the use of the HTTP GET and POST methods defined in RFC 2616 [RFC2616] for each endpoint. Additionally, all flows occur over TLS, and the nonces are not shown.

## Request Transfer Context ID
{: #request-transfer-context-id}

The purpose of this message is for the Client to request the Gateway an identifier for the Transfer Context ID {{?I-D.draft-ietf-satp-core}}. Additionally, the Gateway gets to know the mapping between the ID of the Transfer Context and the ID of the digital asset to be transferred.

The parameters of the message payload consist of the following:

- digital_asset_id REQUIRED: The globally unique identifier for the digital asset located in the origin network to be transferred in a subsequent SATP session.

## Response to Transfer Context ID Request

The purpose of this message is for the Gateway to respond to the Client's request of a Transfer Context ID.

The parameters of the message payload consist of the following:
- success REQUIRED: True/false.
- transfer_context_id REQUIRED: A unique identifier of the application layer context representing the SATP transfer to be initiated.
- digital_asset_id REQUIRED: The globally unique identifier for the digital asset located in the origin network to be transferred in a subsequent SATP session.


## Initiate SATP Session Request
{: #initiate-satp-session-request}

The purpose of this message is for the Client to indicate to the Gateway the intention of initiating a transfer of a digital asset using SATP.

The parameters of the message payload consist of the following:

- transfer_context_id REQUIRED: The unique identifier of the application layer context representing the SATP transfer to be initiated.
- satp_version REQUIRED: The version of the SAT protocol
- from_network_id REQUIRED: The identifier of the network from which the digital asset will be transferred.
- to_network_id REQUIRED: The identifier of the network to which the digital asset will be transferred.
- digital_asset_id REQUIRED: The globally unique identifier for the digital asset located in the origin network.
- asset_profile_id REQUIRED: The globally unique identifier for the asset-profile definition (document) on which the digital asset was issued.
- originator_id REQUIRED: The identity data of the originator entity (person or organization) in the origin network.
- beneficiary_id REQUIRED: The identity data of the beneficiary entity (person or organization) in the destination network.
- originator_pubkey REQUIRED: The public key of the asset owner (originator) in the origin network or system.
- beneficiary_pubkey REQUIRED: The public key of the beneficiary in the destination network.
- payload OPTIONAL: Any additional data to be transferred along with the digital asset.
- client_signature REQUIRED: The Client's ECDSA signature over the message.

## Initiate SATP Session Response
{: #initiate-satp-session-response}

This message is sent by the Gateway in response to a Client's request to initiate a SATP session.

The parameters of the message payload consist of the following:
- success OPTIONAL: True/false.
- transfer_context_id REQUIRED: The unique identifier of the application layer context representing the SATP transfer.
- failure_payload OPTIONAL: In case of failure, the Gateway can return to the client more information about why the SATP session was not initiated.

## Cancel SATP Session Request
{: #cancel-satp-session-request}

The purpose of this message is for the Client to indicate to the Gateway the intention of canceling an ongoing transfer of a digital asset. The cancellation is successful only if the Gateway has not yet committed to transferring the digital asset in the current session.

- transfer_context_id REQUIRED: The unique identifier of the application layer context representing the SATP transfer.
- client_signature REQUIRED: The Client's ECDSA signature over the message.

## Cancel SATP Session Response
{: #cancel-satp-session-response}

The purpose of this message is for the Gateway to indicate to the Client the new state of the digital asset transfer after the cancellation request.

- transfer_context_id REQUIRED: The unique identifier of the application layer context representing the SATP transfer.
- status_response REQUIRED: An object of type StatusResponse that encapsulates the entire state of the SATP transfer.
- gateway_signature REQUIRED: The Gateway's ECDSA signature over the message.

## Get SATP Session Status Request
{: #get-satp-session-request}

The purpose of this message is for the Client to get the status of a previously initiated digital asset transfer.

- transfer_context_id REQUIRED: The unique identifier of the application layer context representing the SATP transfer.
- client_signature REQUIRED: The Client's ECDSA signature over the message.

## Get SATP Session Status Response
{: #get-satp-session-response}

This message is sent by the Gateway to the Client to indicate the last message exchanged between gateways in the SATP session requested. The body of the response is as follows:

- transfer_context_id REQUIRED: The unique identifier of the application layer context representing the SATP transfer.
- status_response REQUIRED: An object of type StatusResponse that encapsulates the entire state of the SATP transfer.
- gateway_signature REQUIRED: The Gateway's ECDSA signature over the message.

## Get Networks Supported Request
{: #get-networks-supported-request}

The purpose of this message is for the Client to get the list of networks the Gateway is connected to. The request does not have a body.

## Get Networks Supported Response
{: #get-networks-supported-response}

This message is sent by the Gateway to the Client in response to the Get Networks Supported Request. The body of the response is as follows:

- networks_ids REQUIRED: A list of the network identifiers (UUIDv2) of every network to which the Gateway is connected.

## Audit Gateway Request
{: #audit-gateway-request}

The purpose of this message is for the Client to audit digital asset transfers previously initiated by the Client. Audits span a period limited by a start and end date. Optionally includes proofs generated in each SATP session.

- transfer_context_ids_list REQUIRED: A list of unique identifiers that identify each SATP asset transfer.
- audit_start_date REQUIRED: Timestamp referring to when the audit period starts.
- audit_end_date REQUIRED: Timestamp referring to when the audit period ends.
- include_proofs OPTIONAL: True/false. The default is false.
- client_signature REQUIRED: The Client's ECDSA signature over the message.

## Audit Gateway Response
{: #audit-gateway-response}

This message represents the response sent by the Gateway to the Client with audit details. The body of the response is as follows:

- satp_sessions REQUIRED: A list with the details of the previous SATP sessions associated with the TransferContextIDs requested by the Client.
- proofs OPTIONAL: A list of payloads representing the proofs generated in each SATP session.
- gateway_signature REQUIRED: The Gateway's ECDSA signature over the message.

## Get Digital Asset Resource Request
{: #get-ditial-asset-resource-request}

One of the functions of a SATP gateway is to facilitate Clients discovering digital asset resources they do not have access to. The fields of the request are as follows:

- resource_urn REQUIRED: The URN of a resource that the Gateway can retrieve from a resource server using API3.

## Get Digital Asset Resource Response
{: #get-ditial-asset-resource-response}

The purpose of this message is for the Gateway to return the Gateway a list of available URLs in which the Client can search for digital assets. This message is a response to the previous Request. The body of the response is as follows:

- resource_urls REQUIRED: list of URLs available to the Client that provide digital assets.

## Ping Request
{: #ping-request}

The purpose of this message is for the Client to get the current status of the Gateway. Namely, the Client may request whether the Gateway is available to process new Client requests. The request does not have a body.

## Ping Response
{: #ping-response}

This message indicates to the Client the availability status of the Gateway. The body of the response is as follows:

- available REQUIRED: True/false. The default is false.

## Integrations Request
{: #integrations-request}
Requests the gateway metadata about each supported blockchain networks, chains, and other systems. This request has no body nor authentication involved.

## Integrations Response
{: #integrations-response}

The body of the response is as follows:

- integrations: A list of Integration objects

# Data Structures Definition

## StatusResponse schema

- status REQUIRED: NOT_FOUND, INVALID, PENDING, DONE, FAILED
- substatus REQUIRED: WAIT_SOURCE_CONFIRMATIONS, WAIT_DESTINATION_TRANSACTION, BRIDGE_NOT_AVAILABLE, CHAIN_NOT_AVAILABLE, ROLLBACK_IN_PROGRESS, UNKNOWN_ERROR, NOT_PROCESSABLE_REFUND_NEEDED
- stage REQUIRED: the number of the stage
- step REQUIRED: A message in the form urn:ietf:satp:msgtype:msg, where `msg` is the last message exchanged between gateways.
- startTime REQUIRED: Timestamp referring to when the asset transfer started.
- originNetwork REQUIRED: The identifier of the network from which the digital asset will be transferred.
- destinationNetwork REQUIRED: The identifier of the network to which the digital asset will be transferred.

## Integration schema

- id REQUIRED: The unique identifier for the network.
- name REQUIRED: The name of the network (e.g., 'Ethereum blockchain').
- type REQUIRED: The type of network (e.g., 'evm', 'fabric').
- environment REQUIRED: The specific network name (e.g., 'mainnet', 'testnet').

# Security Considerations
{: #security-considerations}

We assume a trusted, authenticated, secure, reliable communication channel between the Client and the Gateway (i.e., messages cannot be spoofed and/or altered by an adversary) using TLS/HTTPS [TLS].

--- back
