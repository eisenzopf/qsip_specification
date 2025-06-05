# The QUIC Signaling and Interaction Protocol (QSIP)
# draft-qsip-core-03

|             |                                   |
| :---------- | :-------------------------------- |
| **Version** | 1.0                               |
| **Date**    | 2025-06-02                        |
| **Status**  | Working Draft                     |
| **Expires** | 2025-12-02                        |
| **Authors** | QSIP Working Group (Conceptual)   |

## Status of this Memo

This Internet-Draft is submitted in full conformance with the provisions of BCP 78 and BCP 79.

Internet-Drafts are working documents of the Internet Engineering Task Force (IETF). Note that other groups may also distribute working documents as Internet-Drafts. The list of current Internet-Drafts is at https://datatracker.ietf.org/drafts/current/.

Internet-Drafts are draft documents valid for a maximum of six months and may be updated, replaced, or obsoleted by other documents at any time. It is inappropriate to use Internet-Drafts as reference material or to cite them other than as "work in progress."

## Abstract

The QUIC Signaling and Interaction Protocol (QSIP) provides a single-port, low-latency, secure signaling and media transport system for real-time communications (RTC). It aims to unify the functionality traditionally delivered by disparate protocols like SIP, XMPP, and WebRTC signaling mechanisms, while leveraging the strengths of QUIC for transport. QSIP supports a wide range of communication scenarios, including peer-to-peer calls, group conferences, instant messaging, presence, file sharing, and call center operations, all over a single QUIC connection. This document defines the core QSIP message envelope, message types, semantics, state considerations, media transport integration, and security model.

## Conventions and Terminology

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**", "**NOT RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all capitals, as shown here.

## Table of Contents

- [1. Introduction](#1-introduction)
  - [1.1. Purpose](#11-purpose)
  - [1.2. Design Goals](#12-design-goals)
  - [1.3. Weaknesses of Existing Protocols](#13-weaknesses-of-existing-protocols)
  - [1.4. Supported Scenarios](#14-supported-scenarios)
- [2. Architectural Overview](#2-architectural-overview)
- [3. Transport Layer](#3-transport-layer)
  - [3.1. QUIC](#31-quic)
  - [3.2. Port](#32-port)
  - [3.3. Stream Usage](#33-stream-usage)
- [4. Message Envelope](#4-message-envelope)
  - [4.1. General Format](#41-general-format)
  - [4.2. Envelope Fields](#42-envelope-fields)
  - [4.3. Size Limits](#43-size-limits)
- [5. Message Types](#5-message-types)
  - [5.1. REGISTER](#51-register)
  - [5.2. PRESENCE](#52-presence)
  - [5.3. INVITE](#53-invite)
  - [5.4. ACCEPT](#54-accept)
  - [5.5. REJECT](#55-reject)
  - [5.6. END](#56-end)
  - [5.7. SESSION](#57-session)
  - [5.8. CHANNEL_UPDATE](#58-channel_update)
  - [5.9. MESSAGE](#59-message)
  - [5.10. REACTION](#510-reaction)
  - [5.11. RECEIPT](#511-receipt)
  - [5.12. MESSAGE_ACTION](#512-message_action)
  - [5.13. INFO](#513-info)
  - [5.14. GROUP](#514-group)
  - [5.15. POLL](#515-poll)
  - [5.16. QUEUE](#516-queue)
  - [5.17. CALENDAR](#517-calendar)
  - [5.18. SCHEDULE](#518-schedule)
  - [5.19. ANALYTICS](#519-analytics)
  - [5.20. MODERATION](#520-moderation)
  - [5.21. AUDIT_EVENT](#521-audit_event)
  - [5.22. ERROR](#522-error)
  - [5.23. KEY_EXCHANGE](#523-key_exchange)
  - [5.24. MEDIA_UPDATE](#524-media_update)
  - [5.25. USER_PROFILE](#525-user_profile)
  - [5.26. SYNC](#526-sync)
  - [5.27. PUSH](#527-push)
  - [5.28. SEARCH](#528-search)
  - [5.29. ROSTER](#529-roster)
  - [5.30. CHANNEL](#530-channel)
  - [5.31. BOT](#531-bot)
  - [5.32. VOICEMAIL](#532-voicemail)
  - [5.33. FEDERATION](#533-federation)
  - [5.34. WORKSPACE](#534-workspace)
  - [5.35. STORY](#535-story)
  - [5.36. FORM](#536-form)
  - [5.37. PAYMENT](#537-payment)
  - [5.38. TRANSCRIPT](#538-transcript)
  - [5.39. BACKUP](#539-backup)
  - [5.40. MFA](#540-mfa)
  - [5.41. SUBSCRIPTION](#541-subscription)
  - [5.42. INTERACTIVE](#542-interactive)
  - [5.43. COMMAND](#543-command)
  - [5.44. WEBHOOK](#544-webhook)
  - [5.45. WORKFLOW](#545-workflow)
  - [5.46. MODAL](#546-modal)
  - [5.47. CLIP](#547-clip)
- [6. Session Management](#6-session-management)
  - [6.1. Session Lifecycle](#61-session-lifecycle)
  - [6.2. State Machines](#62-state-machines)
- [7. Media Transport](#7-media-transport)
  - [7.1. Media Encapsulation](#71-media-encapsulation)
  - [7.2. Codec Negotiation](#72-codec-negotiation)
  - [7.3. SFrame End-to-End Encryption](#73-sframe-end-to-end-encryption)
  - [7.4. Simulcast and Scalable Video Coding (SVC)](#74-simulcast-and-scalable-video-coding-svc)
  - [7.5. NAT Traversal](#75-nat-traversal)
- [8. Authentication and Authorization](#8-authentication-and-authorization)
  - [8.1. Device Registration](#81-device-registration)
  - [8.2. Token-based Authentication](#82-token-based-authentication)
  - [8.3. Token Lifecycle](#83-token-lifecycle)
- [9. Error Handling](#9-error-handling)
  - [9.1. ERROR Message Type](#91-error-message-type)
  - [9.2. Error Codes](#92-error-codes)
- [10. Extensibility](#10-extensibility)
  - [10.1. Custom Message Types](#101-custom-message-types)
  - [10.2. Payload Extensions](#102-payload-extensions)
- [11. Performance Considerations](#11-performance-considerations)
  - [11.1. Latency](#111-latency)
  - [11.2. Bandwidth Efficiency](#112-bandwidth-efficiency)
  - [11.3. Congestion Control](#113-congestion-control)
- [12. Security Considerations](#12-security-considerations)
- [13. Privacy Considerations](#13-privacy-considerations)
- [14. IANA Considerations](#14-iana-considerations)
  - [14.1. QSIP Message Type Registry](#141-qsip-message-type-registry)
  - [14.2. QSIP Error Code Registry](#142-qsip-error-code-registry)
  - [14.3. QSIP Info Type Registry](#143-qsip-info-type-registry)
  - [14.4. QSIP Action Type Registries](#144-qsip-action-type-registries)
- [15. Backwards-Compatibility and Interoperability](#15-backwards-compatibility-and-interoperability)
  - [15.1. QSIP-to-SIP Gateway](#151-qsip-to-sip-gateway)
  - [15.2. QSIP-to-XMPP Gateway](#152-qsip-to-xmpp-gateway)
  - [15.3. RTP Fallback Considerations](#153-rtp-fallback-considerations)
- [16. Multi-Device Synchronization](#16-multi-device-synchronization)
  - [16.1. Device Management](#161-device-management)
  - [16.2. Message Synchronization](#162-message-synchronization)
  - [16.3. State Synchronization](#163-state-synchronization)
- [17. Federation](#17-federation)
  - [17.1. Server Discovery](#171-server-discovery)
  - [17.2. Cross-Server Authentication](#172-cross-server-authentication)
  - [17.3. Message Routing](#173-message-routing)
- [18. App Development](#18-app-development)
  - [18.1. App Registration](#181-app-registration)
  - [18.2. Permissions and Scopes](#182-permissions-and-scopes)
  - [18.3. App Distribution](#183-app-distribution)
- [19. JSON Schemas](#19-json-schemas)
- [20. References](#20-references)
  - [20.1. Normative References](#201-normative-references)
  - [20.2. Informative References](#202-informative-references)
- [Appendix A: Example Flows](#appendix-a-example-flows)
- [Appendix B: Detailed State Machines](#appendix-b-detailed-state-machines)
- [Appendix C: Comparative Analysis](#appendix-c-comparative-analysis)
- [Authors' Addresses](#authors-addresses)

---

## 1. Introduction

### 1.1. Purpose

The QUIC Signaling and Interaction Protocol (QSIP) is designed to provide a unified, efficient, and secure framework for real-time communications (RTC) and interactive applications. It aims to simplify the complex landscape of RTC protocols by leveraging QUIC [RFC9000] as its primary transport. QSIP addresses the signaling, messaging, presence, session management, and media orchestration needs for a wide array of use cases, including peer-to-peer (P2P) calls, group conferences, instant messaging, call centers, and collaborative applications.

By consolidating these functions over a single QUIC connection, QSIP intends to reduce connection setup latency, improve NAT traversal, enhance security by default, and provide a more streamlined development experience compared to traditional RTC stacks that often involve multiple protocols (e.g., SIP, XMPP, HTTP, RTP) and ports.

### 1.2. Design Goals

QSIP development is guided by the following primary design goals:

*   **Single Port Operation**: Utilize a single UDP port for all signaling, messaging, and media-related datagrams via QUIC, simplifying network configuration and NAT traversal.
*   **Low Latency**: Achieve significantly faster session setup and message delivery times compared to traditional protocols by leveraging QUIC's 0-RTT/1-RTT handshakes and multiplexing capabilities.
*   **Inherent Security**: Provide robust security by default through QUIC's mandatory TLS 1.3 encryption for all communication. Support strong end-to-end encryption (E2EE) mechanisms like SFrame for media.
*   **Simplicity and Extensibility**: Employ a human-readable JSON-based message envelope and a modular message type system that is easy for developers to implement, understand, and extend. New functionalities and message types can be added via a well-defined registry system.
*   **Comprehensive Feature Set**: Support a rich set of communication features, including voice/video calling, rich messaging (text, files, reactions, threads), presence, group management, call center queueing, scheduling, and moderation, all within a unified protocol.
*   **Efficiency**: Optimize for network bandwidth and server resources through efficient message payloads, modern codecs, and QUIC's advanced transport features.
*   **Interoperability**: Define clear mechanisms for potential interoperability with legacy systems (e.g., SIP, XMPP) through gateways.
*   **Cross-Platform**: Be suitable for implementation across various platforms, including mobile devices, desktop computers, and web browsers (e.g., via WebAssembly or native QUIC APIs).

### 1.3. Weaknesses of Existing Protocols

QSIP aims to address several limitations observed in traditional RTC and messaging protocols:

| Issue                      | SIP / RTP                                 | XMPP                                     | WebRTC (Signaling is undefined)          | QSIP Solution                                                                  |
| :------------------------- | :---------------------------------------- | :--------------------------------------- | :--------------------------------------- | :----------------------------------------------------------------------------- |
| **Multiple Ports/NAT**     | **Yes** (SIP + multiple RTP/RTCP ports)   | Sometimes (core + extensions)            | **Yes** (Signaling + multiple RTP/RTCP)  | **Single UDP port** via QUIC for all streams.                                  |
| **High Handshake Latency** | Yes (TCP/TLS + SIP handshake)             | Moderate (TCP/TLS + XMPP handshake)      | Yes (WebSocket/TLS + SDP negotiation)    | **0-1 RTT** QUIC handshake, multiplexed streams.                               |
| **Fragmented Security**    | Optional TLS for SIP, SRTP for media.     | TLS common, E2EE via OMEMO/OTR optional. | DTLS-SRTP for media, signaling security varies. | **Mandatory TLS 1.3** (via QUIC) for all traffic, explicit SFrame E2EE for media. |
| **No Unified Messaging/Media** | Yes (SIP for signaling, RTP for media)  | Primarily messaging, media via Jingle. | Media-focused, signaling external.       | **Unified protocol** for signaling, messaging, and media transport control.    |
| **Heavy SDP Parsing**      | **Yes**, complex and verbose.             | Jingle uses XML-based descriptions.      | **Yes**, SDP is core.                    | **Simplified media negotiation** (e.g., `MEDIA_UPDATE`), lean JSON payloads.    |
| **Complexity**             | High protocol and state machine complexity. | High extension and stanza complexity.    | Signaling complexity devolved to application. | **Simplified JSON envelope**, clear message types, QUIC handles transport complexity. |

### 1.4. Supported Scenarios

QSIP is designed to support a comprehensive range of modern communication and collaboration scenarios. The following examples demonstrate the protocol's versatility:

#### 1.4.1. Real-Time Communication

* **Peer-to-Peer Audio/Video Calls**: Direct calls between two users with end-to-end encryption via SFrame
* **Group Video Conferences**: Multi-party video meetings with SFU/MCU support, screen sharing, and recording
* **Live Streaming**: One-to-many broadcasting with real-time chat and reactions
* **Instant Audio Rooms**: Quick voice conversations (like Slack Huddles) without formal meeting setup

#### 1.4.2. Messaging and Collaboration

* **Rich Text Messaging**: Text with markdown formatting, mentions, and threading
* **File Sharing**: Documents, images, and media files with progress tracking
* **Ephemeral Messages**: Self-destructing messages with configurable TTL
* **Message Reactions**: Emoji reactions and quick responses
* **Threaded Discussions**: Organized conversations with topic threading
* **Voice/Video Messages**: Asynchronous clips with automatic transcription

#### 1.4.3. Interactive Content

* **Rich Cards**: Structured content with images, buttons, and actions
* **Interactive Forms**: Modal dialogs for data collection and workflows
* **Polls and Surveys**: Real-time voting with result aggregation
* **Collaborative Canvas**: Shared whiteboards and drawing surfaces
* **Embedded Apps**: Mini-applications and widgets within messages

#### 1.4.4. Automation and Integration

* **Slash Commands**: Quick actions like `/remind`, `/poll`, `/call`
* **Webhooks**: Bidirectional integration with external services
* **Automated Workflows**: Multi-step processes with conditional logic
* **Scheduled Messages**: Time-delayed content delivery
* **Bot Interactions**: AI assistants and automated agents
* **Event Subscriptions**: Follow threads, channels, or automated tool outputs

#### 1.4.5. Team Collaboration

* **Workspaces**: Organized team environments with channels and groups
* **Channel Management**: Public/private channels with granular permissions
* **Presence and Status**: Real-time availability with custom messages
* **Calendar Integration**: Meeting scheduling and event management
* **Task Management**: To-do lists and project tracking

#### 1.4.6. Call Center Operations

* **Queue Management**: Intelligent routing with skills-based assignment
* **Agent Monitoring**: Real-time status and performance analytics
* **Call Recording**: Compliance recording with transcription
* **Customer Context**: Screen pops with interaction history
* **Supervisor Features**: Listen-in, whisper, and barge capabilities

#### 1.4.7. Enterprise Features

* **Multi-Device Sync**: Seamless experience across desktop, mobile, and web
* **Federation**: Cross-organization communication with security controls
* **Audit Logging**: Comprehensive activity tracking for compliance
* **Moderation Tools**: Content filtering and user management
* **Backup and Archive**: Message retention and data export

#### 1.4.8. Example Scenario: Collaborative Meeting with Follow-up

This scenario demonstrates how multiple QSIP features work together:

1. **Meeting Setup**: Alice uses `/meeting` command to schedule a team meeting
2. **Video Conference**: Team joins via INVITE/ACCEPT with screen sharing
3. **Collaborative Whiteboard**: During meeting, team uses canvas for brainstorming
4. **File Sharing**: Documents are shared in the meeting chat
5. **Action Items**: Bot captures action items from transcript
6. **Follow-up**: Workflow sends daily reminders until tasks complete
7. **Reporting**: Analytics show meeting effectiveness metrics

```json
// Example flow showing integration of features
COMMAND (slash) → CALENDAR (create) → INVITE (group video) → 
MESSAGE (canvas) → TRANSCRIPT (add) → WORKFLOW (create follow-up) → 
SUBSCRIPTION (daily summaries) → ANALYTICS (report)
```

These scenarios demonstrate QSIP's ability to handle modern communication needs through a single, unified protocol over QUIC transport.

## 2. Architectural Overview

QSIP operates on a client-server model, where clients (endpoints) communicate with a QSIP server, or directly with other clients in P2P scenarios once orchestrated by a server. A QSIP server typically handles user registration, authentication, presence, message routing, group management, and call orchestration. For P2P media, the server may facilitate NAT traversal (e.g., ICE negotiation) and then step out of the media path. For group communication, a Selective Forwarding Unit (SFU) or Multipoint Control Unit (MCU) functionality can be integrated with or act as a QSIP server/endpoint.

```
┌──────────┐ reliable QUIC stream(s) ┌────────────┐ reliable QUIC stream(s) ┌──────────┐
│ Client A │ ◄══════ QSIP Signaling ══════► │ QSIP Server/ │ ◄══════ QSIP Signaling ══════► │ Client B │
│          │                                │     SFU      │                                │          │
│          │ ════ QUIC Datagrams (Media) ═► │              │ ◄════ QUIC Datagrams (Media) ══ │          │
└──────────┘                                └────────────┘                                └──────────┘
                        (Signaling & Media via single QUIC connection to Server/SFU)
```

In a pure P2P media scenario after server-facilitated setup:

```
┌──────────┐ reliable QUIC stream (signaling) ┌────────────┐
│ Client A │ ◄════════════════════════════════► │ QSIP Server│
└──────────┘                                  └────────────┘
                           │
    Reliable QUIC stream (signaling to B via Server, or direct if supported)
                           │
┌─────────────────────────────────────────────────────┐
▼                                                     ▼
┌──────────┐                                      ┌──────────┐
│ Client A │ ═══════ QUIC Datagrams (P2P Media) ════════► │ Client B │
└──────────┘ ◄═══════ QUIC Datagrams (P2P Media) ════════ └──────────┘
                (P2P media over a separate or existing QUIC connection if direct)
```

All interactions, whether signaling messages, text messages, or media datagrams, are multiplexed over a single QUIC connection between an endpoint and its peer (which could be a server or another client).

## 3. Transport Layer

### 3.1. QUIC

QSIP **MUST** be transported over QUIC version 1 [RFC9000] or later compatible versions. QUIC provides a secure, reliable, and low-latency transport layer, incorporating features like stream multiplexing, per-stream flow control, and mandatory TLS 1.3 [RFC8446] encryption.

### 3.2. Port

QSIP services **SHOULD** listen on UDP port **4433** by default. However, deployments **MAY** use other ports if explicitly configured. The use of a distinct default port helps in network identification and firewall configuration.

### 3.3. Stream Usage

QUIC's stream multiplexing capabilities are central to QSIP's design:

*   **Signaling and Control Messages**: QSIP messages (e.g., INVITE, MESSAGE, PRESENCE) **MUST** be sent over reliable, ordered, bidirectional QUIC streams. A client typically opens one primary bidirectional stream to the server for most of its signaling traffic after connection establishment. Additional streams **MAY** be used for concurrent operations or by the server to send messages to the client.
*   **Media Transport**: Real-time media packets (e.g., audio, video) **SHOULD** be transported using QUIC Datagrams [RFC9221] when available and negotiated. This provides low-latency, unordered delivery suitable for media. Each distinct media flow (e.g., one audio track, one video track) **MAY** use its own logical identifier within the datagrams or be multiplexed if higher-layer framing (like RTP) supports it.
*   **File Transfers**: Large file transfers initiated via a `MESSAGE` type with `message_type: "file"` **MAY** be broken down and sent over dedicated reliable QUIC streams for robust delivery, or use an external transfer mechanism referenced by a URL in the message. The specific method for file data transport is outside the strict scope of QSIP signaling but can be orchestrated by it.

The use of a single QUIC connection for both reliable signaling and unreliable media datagrams simplifies NAT traversal and reduces connection overhead.

## 4. Message Envelope

All QSIP messages **MUST** adhere to a common JSON envelope structure. JSON is chosen for its human readability and widespread support in development environments.

### 4.1. General Format

```json
{
  "qsip_version": "1.0",
  "message_id": "UUIDv4",
  "timestamp": "ISO8601-UTC",
  "from": {
    "user_id": "string",
    "device_id": "string"
  },
  "to": {
    "user_id": "string | null",
    "group_id": "string | null",
    "call_center_id": "string | null",
    "server_id": "string | null"
  },
  "type": "MESSAGE_TYPE_STRING",
  "session_id": "UUIDv4 | null",
  "payload": {
    // Message type-specific fields
  }
}
```

### 4.2. Envelope Fields

* **`qsip_version`** (string, REQUIRED): The version of the QSIP protocol. This document defines "1.0". Clients and servers MUST include this field.
* **`message_id`** (string, REQUIRED): A unique identifier for this message, typically a UUIDv4. Used for tracking, ACKs, and de-duplication.
* **`timestamp`** (string, REQUIRED): An ISO 8601 formatted UTC timestamp indicating when the message was generated by the sender (e.g., "2025-06-01T00:43:25Z").
* **`from`** (object, REQUIRED): Identifies the originator of the message.
  * **`user_id`** (string, REQUIRED): A unique identifier for the sending user.
  * **`device_id`** (string, REQUIRED): A unique identifier for the sending device of that user.
  * **`display_name`** (string, OPTIONAL): The display name of the sending user, primarily for informational purposes.
* **`to`** (object, REQUIRED): Identifies the recipient(s) of the message. At least one of `user_id`, `group_id`, `call_center_id`, or `server_id` SHOULD be present, unless the message type inherently implies a server-wide scope (e.g., initial `REGISTER` to the connected server). For messages explicitly to the server (e.g. `REGISTER`, some `PRESENCE` updates), `to` can be an empty object `{}` or contain a `server_id`.
  * **`user_id`** (string | null, OPTIONAL): The unique identifier of the target user.
  * **`group_id`** (string | null, OPTIONAL): The unique identifier of a target group or chat room.
  * **`call_center_id`** (string | null, OPTIONAL): The unique identifier of a target call center queue or service.
  * **`server_id`** (string | null, OPTIONAL): The unique identifier of the target server, if applicable (e.g., in multi-server federated environments or for messages explicitly to the server itself).
* **`type`** (string, REQUIRED): The QSIP message type (e.g., "INVITE", "MESSAGE"). See Section 5 for defined types. This field is case-sensitive and MUST be one of the registered types.
* **`session_id`** (string | null, OPTIONAL): A unique identifier for a specific communication session (e.g., a call, a conference). This field MUST be present for messages related to an ongoing session (e.g., `INVITE`, `ACCEPT`, `REJECT`, `END`, `SESSION`, `CHANNEL_UPDATE`, `KEY_EXCHANGE`, `MEDIA_UPDATE`). For other messages, it MAY be null or omitted.
* **`payload`** (object, REQUIRED): An object containing data specific to the message type. The structure of the payload varies per message type. If a message type has no specific data, the payload MUST be an empty object `{}`.

Endpoints MUST ignore any unrecognized top-level keys in the JSON envelope to allow for forward compatibility.

### 4.3. Size Limits

Individual QSIP messages sent over reliable QUIC streams SHOULD NOT exceed 64 KiB (65536 bytes) when serialized as UTF-8 JSON. Messages exceeding this limit MAY be rejected by the recipient with an `ERROR` message (see Section 9). This limit helps prevent excessive buffer allocation and stream blocking. Larger data, such as files, SHOULD be transferred using mechanisms suitable for large binary objects (e.g., dedicated QUIC streams or out-of-band HTTP transfers referenced by a URL).

## 5. Message Types

This section defines the standard QSIP message types and their payloads.

### 5.1. REGISTER

**Purpose**: Allows a client device to authenticate with the server, register its presence, and declare its capabilities. This is typically the first message sent by a client after establishing a QUIC connection.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`auth_token`** (string, REQUIRED): An authentication token (e.g., JWT, API key) used to verify the client's identity. The specifics of token acquisition are outside QSIP but often involve an external identity provider or OAuth flow.
* **`device_info`** (object, REQUIRED): Information about the client device.
  * **`os`** (string, OPTIONAL): Operating system (e.g., "Android", "iOS", "Windows", "macOS", "Linux").
  * **`app`** (string, OPTIONAL): Application name and version (e.g., "qsip-mobile 3.0", "qsip-desktop 1.2.0").
  * **`device_model`** (string, OPTIONAL): Specific device model (e.g., "Pixel 8", "iPhone 15 Pro").
* **`capabilities`** (array of strings, OPTIONAL): A list of features or codecs supported by the client (e.g., ["av1", "opus", "sframe", "simulcast", "h264_baseline"]). This helps the server and peers understand what the client can handle.
* **`supports_datagrams`** (boolean, OPTIONAL): Indicates if the client supports QUIC datagrams for media. Defaults to false if not present.

**Normative Requirements**:

* A client MUST send a `REGISTER` message to the server before attempting to send most other QSIP messages.
* The server MUST validate the `auth_token`.
* Upon successful registration, the server SHOULD reply with a `RECEIPT` message with `status: "registered"` or a more specific positive acknowledgment.
* If registration fails, the server MUST reply with an `ERROR` message.

**Example**:

```json
{
  "qsip_version": "1.0",
  "message_id": "7be01437-0cfe-443c-b49d-fae161750009",
  "timestamp": "2025-06-01T00:43:25Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {},
  "type": "REGISTER",
  "payload": {
    "auth_token": "jwt-access-token-for-alice",
    "device_info": {
      "os": "Android",
      "app": "qsip-mobile 3.0",
      "device_model": "Pixel 8"
    },
    "capabilities": [
      "av1",
      "opus",
      "sframe",
      "simulcast"
    ],
    "supports_datagrams": true
  }
}
```

### 5.2. PRESENCE

**Purpose**: Used by clients to publish their current availability status and activity to the server, which can then distribute this information to subscribed contacts.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`status`** (string, REQUIRED): The user's availability state. Common values include:
  * `"available"`: User is online and ready to communicate.
  * `"away"`: User is online but temporarily inactive.
  * `"busy"`: User is online but currently engaged (e.g., in a call, meeting).
  * `"dnd"`: (Do Not Disturb) User is online but prefers not to be interrupted.
  * `"offline"`: User is not connected. This can be explicitly set or inferred by the server on disconnection.
* **`status_message`** (string, OPTIONAL): A custom message accompanying the status (e.g., "In a meeting until 3 PM", "Focusing").
* **`activity`** (string, OPTIONAL): Describes the user's current activity. Common values:
  * `"idle"`: No specific activity.
  * `"typing"`: User is typing (often part of an `INFO` message, but can be part of general presence).
  * `"in_call"`: User is in an audio/video call.
  * `"in_meeting"`: User is in a scheduled meeting.
  * `"presenting"`: User is sharing their screen or presenting.
* **`last_active_at`** (string, OPTIONAL): ISO 8601 timestamp of the user's last known activity, useful for "away" status.

**Normative Requirements**:

* A client SHOULD send a `PRESENCE` update after successful registration.
* Clients SHOULD send `PRESENCE` updates whenever their status or activity changes significantly.
* Servers SHOULD manage presence state and distribute updates to authorized subscribers (e.g., contacts in a roster).

**Example**:

```json
{
  "qsip_version": "1.0",
  "message_id": "1a2b3c4d-5e6f-7g8h-9i0j-klmnopqrstuv",
  "timestamp": "2025-06-01T00:45:10Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {},
  "type": "PRESENCE",
  "payload": {
    "status": "busy",
    "status_message": "On an important call",
    "activity": "in_call"
  }
}
```

### 5.3. INVITE

**Purpose**: Initiates a new communication session, such as a voice/video call or a conference. It typically contains media descriptions and NAT traversal candidates.

**`session_id`**: REQUIRED. This session_id will be used for all subsequent messages within this session.

**Payload Schema**:

* **`media_types`** (array of strings, REQUIRED): Specifies the types of media offered for the session (e.g., `["audio", "video"]`, `["audio"]`, `["screenshare"]`).
* **`codecs`** (object, OPTIONAL): Details the codecs offered by the initiator for each media type.
  * **`audio`** (array of strings, OPTIONAL): e.g., `["opus/48000/2"]` (Opus, 48kHz, stereo).
  * **`video`** (array of strings, OPTIONAL): e.g., `["av1/90000", "h264_constrained_baseline/90000"]`. Clock rates (e.g., 90000 for video) SHOULD be included.
* **`sdp_offer`** (string, OPTIONAL): A Session Description Protocol (SDP) offer, typically Base64 encoded or in a minified JSON format if a QSIP-specific SDP-lite is used. This is for compatibility with WebRTC peers or more complex media negotiation.
* **`ice_candidates`** (array of objects, OPTIONAL): A list of ICE candidates for NAT traversal. Each object might contain:
  * **`candidate`** (string): The SDP a=candidate line value.
  * **`sdpMid`** (string): Media stream identifier.
  * **`sdpMLineIndex`** (number): Media line index.
* **`encryption`** (array of strings, OPTIONAL): Proposed E2EE mechanisms, e.g., `["sframe"]`.
* **`target_group_id`** (string, OPTIONAL): If inviting to a group session managed by an SFU/MCU, this specifies the group.
* **`purpose`** (string, OPTIONAL): A hint about the call's purpose, e.g., `"customer_support_call"`, `"team_sync"`.

**Normative Requirements**:

* The `to.user_id` (for P2P calls) or `to.group_id` / `payload.target_group_id` (for group calls) MUST be specified.
* The initiator MUST generate a unique `session_id` for the new session.
* The recipient(s) MUST respond with `ACCEPT`, `REJECT`, or an `ERROR` message.

**Example**:

```json
{
  "qsip_version": "1.0",
  "message_id": "fdee98d1-fcbf-4def-b7d6-ebbd2d6718b0",
  "timestamp": "2025-06-01T00:50:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {
    "user_id": "u-bob"
  },
  "type": "INVITE",
  "session_id": "s-call-alice-bob-123",
  "payload": {
    "media_types": [
      "audio",
      "video"
    ],
    "codecs": {
      "audio": [
        "opus/48000/2"
      ],
      "video": [
        "av1/90000",
        "vp9/90000"
      ]
    },
    "sdp_offer": "base64-encoded-webrtc-sdp-offer-string",
    "ice_candidates": [
      {
        "candidate": "candidate:1 1 udp 2122260223 192.0.2.1 60000 typ host generation 0",
        "sdpMid": "audio",
        "sdpMLineIndex": 0
      },
      {
        "candidate": "candidate:2 1 udp 2122260222 192.0.2.1 60002 typ host generation 0",
        "sdpMid": "video",
        "sdpMLineIndex": 1
      }
    ],
    "encryption": ["sframe"]
  }
}
```

### 5.4. ACCEPT

**Purpose**: Sent by a recipient to accept an incoming INVITE for a session.

**`session_id`**: REQUIRED. Must match the session_id from the INVITE.

**Payload Schema**:

* **`codecs`** (object, OPTIONAL): The codecs selected by the acceptor from the offerer's list. Structure is the same as in INVITE.
* **`sdp_answer`** (string, OPTIONAL): An SDP answer, typically Base64 encoded, corresponding to the sdp_offer in the INVITE.
* **`ice_candidates`** (array of objects, OPTIONAL): The acceptor's ICE candidates. Structure is the same as in INVITE.
* **`encryption`** (string, OPTIONAL): The selected E2EE mechanism, e.g., "sframe".

**Normative Requirements**:

* MUST be sent in response to an INVITE if the session is accepted.
* The from and to fields are typically reversed from the INVITE.

**Example**:

```json
{
  "qsip_version": "1.0",
  "message_id": "a1b2c3d4-e5f6-g7h8-i9j0-k1l2m3n4o5p6",
  "timestamp": "2025-06-01T00:50:05Z",
  "from": {
    "user_id": "u-bob",
    "device_id": "d-bob-desktop"
  },
  "to": {
    "user_id": "u-alice"
  },
  "type": "ACCEPT",
  "session_id": "s-call-alice-bob-123",
  "payload": {
    "codecs": {
      "audio": [
        "opus/48000/2"
      ],
      "video": [
        "av1/90000"
      ]
    },
    "sdp_answer": "base64-encoded-webrtc-sdp-answer-string",
    "ice_candidates": [
      {
        "candidate": "candidate:3 1 udp 2122187007 198.51.100.5 60008 typ host generation 0",
        "sdpMid": "audio",
        "sdpMLineIndex": 0
      }
    ],
    "encryption": "sframe"
  }
}
```

### 5.5. REJECT

**Purpose**: Sent by a recipient to decline an incoming INVITE.

**`session_id`**: REQUIRED. Must match the session_id from the INVITE.

**Payload Schema**:

* **`reason`** (string, REQUIRED): A reason code for rejection. Common values:
  * `"busy"`: User is busy.
  * `"decline"`: User actively declined.
  * `"unavailable"`: User is not available.
  * `"unsupported"`: Requested media or capabilities are not supported.
  * `"error"`: An internal error occurred.
* **`detail`** (string, OPTIONAL): A human-readable explanation for the rejection.

**Normative Requirements**:

* MUST be sent in response to an INVITE if the session is rejected.

**Example**:

```json
{
  "qsip_version": "1.0",
  "message_id": "b2c3d4e5-f6g7-h8i9-j0k1-l2m3n4o5p6q7",
  "timestamp": "2025-06-01T00:50:03Z",
  "from": {
    "user_id": "u-bob",
    "device_id": "d-bob-desktop"
  },
  "to": {
    "user_id": "u-alice"
  },
  "type": "REJECT",
  "session_id": "s-call-alice-bob-123",
  "payload": {
    "reason": "busy",
    "detail": "Currently in another call."
  }
}
```

### 5.6. END

**Purpose**: Terminates an active communication session.

**`session_id`**: REQUIRED. Must match the session_id of the session to terminate.

**Payload Schema**:

* **`reason`** (string, REQUIRED): A reason code for ending the session. Common values:
  * `"hangup"`: Normal hangup by a participant.
  * `"completed"`: Session completed successfully (e.g., file transfer finished).
  * `"timeout"`: Session timed out due to inactivity.
  * `"error"`: Session terminated due to an error.
  * `"declined_media"`: Session ended because media negotiation failed post-setup.
* **`detail`** (string, OPTIONAL): A human-readable explanation.

**Normative Requirements**:

* Can be sent by any participant in the session or by the server.

**Example**:

```json
{
  "qsip_version": "1.0",
  "message_id": "c3d4e5f6-g7h8-i9j0-k1l2-m3n4o5p6q7r8",
  "timestamp": "2025-06-01T01:15:30Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {
    "user_id": "u-bob"
  },
  "type": "END",
  "session_id": "s-call-alice-bob-123",
  "payload": {
    "reason": "hangup",
    "detail": "User ended the call."
  }
}
```

### 5.7. SESSION

**Purpose**: Used for mid-session control actions, such as putting a call on hold, resuming, muting/unmuting, or initiating a transfer.

**`session_id`**: REQUIRED.

**Payload Schema**:

* **`action`** (string, REQUIRED): The session control action to perform. Examples:
  * `"hold"`: Place the session on hold.
  * `"resume"`: Resume a session that was on hold.
  * `"mute"`: Mute a specific media stream (e.g., audio).
  * `"unmute"`: Unmute a specific media stream.
  * `"start_screenshare"`: Start sharing screen.
  * `"stop_screenshare"`: Stop sharing screen.
  * `"start_recording"`: Start recording the session.
  * `"stop_recording"`: Stop recording the session.
  * `"transfer"`: Initiate a call transfer.
* **`state`** (string, OPTIONAL): The resulting state after the action, if applicable (e.g., "on_hold", "recording").
* **`target_media_type`** (string, OPTIONAL): For actions like mute/unmute, specifies the media type (e.g., "audio", "video").
* **`transfer_to_user_id`** (string, OPTIONAL): For transfer action, the user_id to transfer the call to.
* **`stream_id`** (string, OPTIONAL): For start_screenshare, an identifier for the new media stream.

**Normative Requirements**:

* The action MUST be one of the defined session actions.

**Example (Hold Call)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "d4e5f6g7-h8i9-j0k1-l2m3-n4o5p6q7r8s9",
  "timestamp": "2025-06-01T01:05:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {
    "user_id": "u-bob"
  },
  "type": "SESSION",
  "session_id": "s-call-alice-bob-123",
  "payload": {
    "action": "hold",
    "state": "on_hold"
  }
}
```

**Example (Start Screenshare)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "e5f6g7h8-i9j0-k1l2-m3n4-o5p6q7r8s9t0",
  "timestamp": "2025-06-01T01:08:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {
    "user_id": "u-bob"
  },
  "type": "SESSION",
  "session_id": "s-call-alice-bob-123",
  "payload": {
    "action": "start_screenshare",
    "stream_id": "screen-stream-alice-1",
    "state": "screensharing"
  }
}
```

### 5.8. CHANNEL_UPDATE

**Purpose**: Informs other participants or the server about a change in the sender's media channel status or capabilities during an active session (e.g., camera turned on/off, microphone muted/unmuted).

**`session_id`**: REQUIRED.

**Payload Schema**:

* **`channels`** (object, REQUIRED): An object where keys are media types (e.g., "audio", "video", "screenshare") and values are objects describing the channel's state.
  * **`<media_type>`** (object):
    * **`enabled`** (boolean, OPTIONAL): Indicates if the channel is actively being sent/received (e.g., camera is on).
    * **`muted`** (boolean, OPTIONAL): For "audio", indicates if the microphone is muted.
    * **`available`** (boolean, OPTIONAL): Indicates if the device/source for this channel is physically available (e.g., camera connected).
    * **`active`** (boolean, OPTIONAL): Indicates if the channel is currently active (e.g. screen sharing in progress).

**Normative Requirements**:

* Clients SHOULD send CHANNEL_UPDATE when a user manually changes a media device's state.

**Example (User disabled video camera)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "f6g7h8i9-j0k1-l2m3-n4o5-p6q7r8s9t0u1",
  "timestamp": "2025-06-01T01:10:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {
    "user_id": "u-bob" // Or to group/server if in a conference
  },
  "type": "CHANNEL_UPDATE",
  "session_id": "s-call-alice-bob-123",
  "payload": {
    "channels": {
      "video": {
        "enabled": false,
        "available": true // Camera is still there, just turned off
      }
    }
  }
}
```

### 5.9. MESSAGE

**Purpose**: Used to send chat messages (text, files, rich content) either peer-to-peer or within a group.

**`session_id`**: null or omitted (unless the message is contextually tied to an active call session).

**Payload Schema**:

* **`message_type`** (string, REQUIRED): The type of message content. Common values:
  * `"text"`: Plain text or markdown-formatted text.
  * `"file"`: A file attachment.
  * `"image"`: An image (can be a specialization of "file").
  * `"audio_clip"`: A voice message (can be a specialization of "file").
  * `"video_clip"`: A short video message (can be a specialization of "file").
  * `"location"`: A geographical location.
  * `"card"`: A rich card with structured content.
  * `"carousel"`: A carousel of multiple cards or media items.
  * `"canvas"`: A collaborative whiteboard or drawing canvas.
  * `"embed"`: An embedded app or widget.
* **`content`** (string or object, REQUIRED): The actual message content.
  * For `"text"`: A string containing the text. Markdown SHOULD be supported for rich text formatting.
  * For `"file"`, "image", "audio_clip", "video_clip": An object with details like:
    * **`file_name`** (string, REQUIRED): Original name of the file.
    * **`file_size`** (number, REQUIRED): Size of the file in bytes.
    * **`mime_type`** (string, REQUIRED): MIME type of the file (e.g., "application/pdf", "image/jpeg").
    * **`url`** (string, REQUIRED): A URL from which the file can be downloaded. QSIP itself does not transport the file data in this message.
    * **`file_hash`** (string, OPTIONAL): A hash of the file (e.g., "sha256:abc...").
    * **`thumbnail_url`** (string, OPTIONAL): For images/videos, a URL to a thumbnail.
    * **`duration_ms`** (number, OPTIONAL): For audio/video clips, duration in milliseconds.
  * For `"location": An object with latitude, longitude, and optionally accuracy, address.
  * For `"card"`: An object with structured card content:
    * **`title`** (string, OPTIONAL): Card title.
    * **`subtitle`** (string, OPTIONAL): Card subtitle.
    * **`body`** (string, OPTIONAL): Main card content (supports markdown).
    * **`image_url`** (string, OPTIONAL): Card header image.
    * **`actions`** (array of objects, OPTIONAL): Interactive buttons/actions. Each object:
      * **`type`** (string, REQUIRED): Action type (e.g., "button", "link").
      * **`label`** (string, REQUIRED): Display text.
      * **`action_id`** (string, REQUIRED): Unique identifier for the action.
      * **`url`** (string, OPTIONAL): For link actions.
      * **`payload`** (object, OPTIONAL): Custom data for the action.
  * For `"carousel"`: An object with:
    * **`items`** (array of objects, REQUIRED): Array of card objects or media items.
    * **`layout`** (string, OPTIONAL): Layout style (e.g., "horizontal", "vertical").
  * For `"canvas"`: An object with:
    * **`canvas_id`** (string, REQUIRED): Unique identifier for the canvas session.
    * **`canvas_url`** (string, OPTIONAL): URL to access the canvas.
    * **`preview_url`** (string, OPTIONAL): Static preview image of current canvas state.
    * **`collaborative`** (boolean, OPTIONAL): Whether multiple users can edit simultaneously.
  * For `"embed"`: An object with:
    * **`embed_type`** (string, REQUIRED): Type of embed (e.g., "iframe", "widget", "mini-app").
    * **`embed_url`** (string, REQUIRED): URL of the embedded content.
    * **`height`** (number, OPTIONAL): Suggested height in pixels.
    * **`width`** (number, OPTIONAL): Suggested width in pixels.
    * **`permissions`** (array of strings, OPTIONAL): Required permissions (e.g., ["camera", "microphone"]).
* **`thread_id`** (string, OPTIONAL): If this message is part of a thread, the message_id of the root message of the thread.
* **`reply_to_message_id`** (string, OPTIONAL): If this message is a direct reply to another message (not necessarily threaded), its message_id.
* **`mentions`** (array of objects, OPTIONAL): List of users mentioned in the message. Each object:
  * **`user_id`** (string, REQUIRED): The ID of the mentioned user.
  * **`display_text`** (string, REQUIRED): The text used for the mention (e.g., "@bob").
* **`schedule_time`** (string, OPTIONAL): ISO 8601 timestamp if this message is to be delivered or made visible at a future time. See also SCHEDULE message type for more robust scheduling.
* **`ephemeral_ttl`** (number, OPTIONAL): Time-to-live in seconds for ephemeral/disappearing messages. After this duration from delivery, the message should be automatically deleted on all devices.
* **`priority`** (string, OPTIONAL): Message priority level. Common values:
  * `"urgent"`: High priority, may trigger special notifications.
  * `"normal"`: Default priority.
  * `"low"`: Low priority, may be batched or delayed.
* **`attachments`** (array of objects, OPTIONAL): Additional attachments beyond the primary content. Each object follows the same structure as file content objects.

**Example (Text Message)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "0e8e1c9f-73e2-4b2b-9c8b-7d4a9f0c1a2b",
  "timestamp": "2025-06-01T01:20:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {
    "user_id": "u-bob"
  },
  "type": "MESSAGE",
  "payload": {
    "message_type": "text",
    "content": "Hello QSIP! This is a **bold** move for _RTC_.\nCheck out `code`.",
    "ephemeral_ttl": 86400,
    "priority": "normal",
    "attachments": []
  }
}
```

**Example (File Message)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "1f9e2d0a-84f3-5c3c-ad9c-8e5b0a1d2c3d",
  "timestamp": "2025-06-01T01:21:00Z",
  "from": {
    "user_id": "u-bob",
    "device_id": "d-bob-desktop"
  },
  "to": {
    "group_id": "g-team-project"
  },
  "type": "MESSAGE",
  "payload": {
    "message_type": "file",
    "content": {
      "file_name": "design-spec-v2.pdf",
      "file_size": 1048576,
      "mime_type": "application/pdf",
      "file_hash": "sha256:abcdef1234567890...",
      "url": "https://cdn.example.com/files/design-spec-v2.pdf"
    },
    "thread_id": "m-root-design-thread-42",
    "ephemeral_ttl": 86400,
    "priority": "normal",
    "attachments": []
  }
}
```

**Example (Card Message)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "2a9f3d1b-95e4-4c3c-bd8c-9e6b1a2d3c4d",
  "timestamp": "2025-06-01T01:22:00Z",
  "from": {
    "user_id": "u-bot-assistant",
    "device_id": "d-bot-server"
  },
  "to": {
    "user_id": "u-alice"
  },
  "type": "MESSAGE",
  "payload": {
    "message_type": "card",
    "content": {
      "title": "Meeting Reminder",
      "subtitle": "Team Standup in 15 minutes",
      "body": "Don't forget about today's standup meeting. We'll be discussing:\n- Sprint progress\n- Blockers\n- Today's goals",
      "image_url": "https://cdn.example.com/meeting-reminder.png",
      "actions": [
        {
          "type": "button",
          "label": "Join Meeting",
          "action_id": "join-meeting-123",
          "payload": { "meeting_id": "standup-2025-06-01" }
        },
        {
          "type": "button",
          "label": "Snooze 5 min",
          "action_id": "snooze-reminder",
          "payload": { "duration": 300 }
        }
      ]
    }
  }
}
```

**Example (Carousel Message)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "3b0a4e2c-06c5-5d4d-ce9e-1b8d3c5f6a7f",
  "timestamp": "2025-06-01T01:23:00Z",
  "from": {
    "user_id": "u-bot-shop",
    "device_id": "d-bot-server"
  },
  "to": {
    "group_id": "g-team-announcements"
  },
  "type": "MESSAGE",
  "payload": {
    "message_type": "carousel",
    "content": {
      "layout": "horizontal",
      "items": [
        {
          "title": "New Feature: Voice Notes",
          "body": "Record and send voice messages directly in chat",
          "image_url": "https://cdn.example.com/feature-voice.png",
          "actions": [
            { "type": "button", "label": "Try it now", "action_id": "try-voice", "payload": {} }
          ]
        },
        {
          "title": "Canvas Collaboration",
          "body": "Draw and brainstorm together in real-time",
          "image_url": "https://cdn.example.com/feature-canvas.png",
          "actions": [
            { "type": "button", "label": "Start Canvas", "action_id": "new-canvas", "payload": {} }
          ]
        }
      ]
    }
  }
}
```

### 5.10. REACTION

**Purpose**: Allows users to add an emoji reaction to a previously sent message.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`target_message_id`** (string, REQUIRED): The message_id of the message being reacted to.
* **`reaction`** (string, REQUIRED): The emoji character(s) representing the reaction (e.g., "👍", "❤️", "🚀").
* **`action`** (string, OPTIONAL, default: "add"): Specifies whether to "add" or "remove" the reaction.

**Example**:

```json
{
  "qsip_version": "1.0",
  "message_id": "2a0f3e1b-95a4-6d4d-be0d-9f6c1b2e3f4e",
  "timestamp": "2025-06-01T01:22:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": { // Can be to user or group where original message resides
    "user_id": "u-bob"
  },
  "type": "REACTION",
  "payload": {
    "target_message_id": "0e8e1c9f-73e2-4b2b-9c8b-7d4a9f0c1a2b", // ID of "Hello QSIP!" message
    "reaction": "👍",
    "action": "add"
  }
}
```

### 5.11. RECEIPT

**Purpose**: Provides acknowledgment for message delivery, read status, or other events.

**`session_id`**: null or omitted (unless acknowledging a message within a specific call session).

**Payload Schema**:

* **`target_message_id`** (string, REQUIRED): The message_id of the message this receipt refers to.
* **`status`** (string, REQUIRED): The status of the receipt. Common values:
  * `"sent"`: Message successfully sent from originating client to its server.
  * `"delivered_to_server"`: Message successfully delivered to recipient's server.
  * `"delivered_to_client"`: Message delivered to one of the recipient's devices.
  * `"read"`: Message read by the recipient user on at least one device.
  * `"failed"`: Message delivery failed.
  * `"registered"`: Special status for acknowledging a REGISTER message.
  * (Other custom statuses related to specific message types, e.g. poll_voted).
* **`detail`** (string, OPTIONAL): Additional details, especially for "failed" status.

**Example (Read Receipt)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "3b1a4f2c-06b5-7e5e-cf1e-0a7d2c3f4a5f",
  "timestamp": "2025-06-01T01:23:00Z",
  "from": {
    "user_id": "u-bob",
    "device_id": "d-bob-desktop"
  },
  "to": {
    "user_id": "u-alice"
  },
  "type": "RECEIPT",
  "payload": {
    "target_message_id": "0e8e1c9f-73e2-4b2b-9c8b-7d4a9f0c1a2b",
    "status": "read"
  }
}
```

### 5.12. MESSAGE_ACTION

**Purpose**: Allows users to perform actions on their previously sent messages, such as editing or deleting them.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The action to perform.
  * `"edit"`: Edit an existing message.
  * `"delete"`: Delete an existing message.
* **`target_message_id`** (string, REQUIRED): The message_id of the message to act upon.
* **`new_content`** (string, OPTIONAL): For "edit" action, the new content of the message (if it was a text message). For other types, this might be a more complex object.
* **`new_payload`** (object, OPTIONAL): For "edit" action, if the entire payload needs to be replaced (e.g., changing file URL or rich content).

**Example (Edit Message)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "4c2b5a3d-17c6-8f6f-df2f-1b8e3d4a5b6a",
  "timestamp": "2025-06-01T01:25:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": { // Target is context of original message (user or group)
    "user_id": "u-bob"
  },
  "type": "MESSAGE_ACTION",
  "payload": {
    "action": "edit",
    "target_message_id": "0e8e1c9f-73e2-4b2b-9c8b-7d4a9f0c1a2b",
    "new_content": "Hello QSIP! This is an *updated* and **bold** move for _RTC_."
  }
}
```

### 5.13. INFO

**Purpose**: Used to convey transient, auxiliary information, such as typing indicators.

**`session_id`**: null or omitted (typing indicators are usually not tied to a call session).

**Payload Schema**:

* **`info_type`** (string, REQUIRED): The type of information being conveyed.
  * `"typing_indicator"`: Indicates typing status.
  * (Other types can be defined, e.g., "user_viewing_message").
* **`is_typing`** (boolean, OPTIONAL): For "typing_indicator", true if the user started typing, false if they stopped.
* **`target_context_id`** (string, OPTIONAL): The ID of the context this info pertains to (e.g., user_id for a 1-1 chat, group_id for a group chat). The to field usually covers this.

**Example (User Started Typing)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "5d3c6b4e-28d7-9a7a-ea3a-2c9f4e5b6c7b",
  "timestamp": "2025-06-01T01:26:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {
    "user_id": "u-bob"
  },
  "type": "INFO",
  "payload": {
    "info_type": "typing_indicator",
    "is_typing": true
  }
}
```

### 5.14. GROUP

**Purpose**: Manages groups or chat rooms (e.g., create, update, add/remove members, manage roles).

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The group management action.
  * `"create"`: Create a new group.
  * `"update"`: Update group metadata (name, topic, avatar).
  * `"delete"`: Delete a group.
  * `"invite"`: Invite users to a group.
  * `"join"`: Request to join a group (if public or requires approval).
  * `"leave"`: Leave a group.
  * `"kick"`: Remove a member from a group.
  * `"set_role"`: Set a member's role (e.g., admin, moderator).
  * `"get_info"`: Request group information.
  * `"list_members"`: Request list of group members.
* **`group_id`** (string, OPTIONAL/REQUIRED depending on action): The ID of the group. REQUIRED for most actions except create.
* **`name`** (string, OPTIONAL): For create/update, the group's name.
* **`topic`** (string, OPTIONAL): For create/update, the group's topic or description.
* **`avatar_url`** (string, OPTIONAL): For create/update, URL to the group's avatar.
* **`members`** (array of strings, OPTIONAL): For create/invite, list of user_ids.
* **`target_user_id`** (string, OPTIONAL): For kick/set_role, the user_id being acted upon.
* **`role`** (string, OPTIONAL): For set_role, the role to assign (e.g., "admin", "member", "moderator").
* **`visibility`** (string, OPTIONAL): For create, e.g., "private", "public".
* **`workspace_id`** (string, OPTIONAL): For workspace-based deployments, the workspace this group belongs to.

**Example (Create Group)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "6e4d7c5f-39e8-0b8b-fb4b-3da05f6c7d8c",
  "timestamp": "2025-06-01T01:30:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To server
  "type": "GROUP",
  "payload": {
    "action": "create",
    "group_id": "g-new-dev-team-alpha", // Client can suggest, server may override/confirm
    "name": "Dev Team Alpha",
    "topic": "Discussing next-gen QSIP features",
    "members": [
      "u-alice",
      "u-bob"
    ],
    "visibility": "private",
    "workspace_id": "w-team-project"
  }
}
```

### 5.15. POLL

**Purpose**: Create, vote on, or manage polls within a chat context (P2P or group).

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED):
  * `"create"`: Create a new poll.
  * `"vote"`: Cast a vote on an existing poll.
  * `"get_results"`: Request current results of a poll.
  * `"close"`: Close a poll to further voting.
* **`poll_id`** (string, OPTIONAL/REQUIRED): Unique ID for the poll. REQUIRED for vote, get_results, close. Client may suggest for create.
* **`question`** (string, OPTIONAL): For create, the poll question.
* **`options`** (array of strings, OPTIONAL): For create, list of poll options.
* **`selected_option_index`** (number, OPTIONAL): For vote, the 0-based index of the selected option.
* **`expires_at`** (string, OPTIONAL): For create, ISO 8601 timestamp when the poll automatically closes.
* **`context_id`** (string, REQUIRED for create): The user_id (for P2P chat polls) or group_id where the poll is created.
* **`results`** (object, OPTIONAL, usually in server response): For get_results or server-pushed updates, an object mapping option strings/indices to vote counts.

**Example (Create Poll)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "7f5e8d6a-4af9-1c9c-ac5c-4eb16a7d8e9d",
  "timestamp": "2025-06-01T01:35:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {
    "group_id": "g-new-dev-team-alpha"
  },
  "type": "POLL",
  "payload": {
    "action": "create",
    "poll_id": "p-logo-choice-001",
    "question": "Which logo concept do you prefer for QSIP v2?",
    "options": [
      "Concept A (Blue Rocket)",
      "Concept B (Green Globe)",
      "Concept C (Purple Quasar)"
    ],
    "expires_at": "2025-06-08T01:35:00Z",
    "context_id": "g-new-dev-team-alpha"
  }
}
```

### 5.16. QUEUE

**Purpose**: Manages interactions with a call center or task queue system.

**`session_id`**: null or omitted initially; a session might be established once connected to an agent.

**Payload Schema**:

* **`action`** (string, REQUIRED): The queue action.
  * `"enqueue"`: Customer requests to enter a queue.
  * `"dequeue"`: Customer requests to leave a queue, or agent removes item.
  * `"get_status"`: Customer requests their current position/wait time.
  * `"assign_agent"`: Server assigns an agent to a queued item (typically server-to-agent).
  * `"agent_accept"`: Agent accepts a queued item.
  * `"agent_reject"`: Agent rejects a queued item.
  * `"update_priority"`: Change priority of a queued item.
* **`queue_id`** (string, REQUIRED): Identifier of the target queue (e.g., "support_technical", "sales_inquiries").
* **`customer_id`** (string, OPTIONAL): For enqueue, the user_id of the customer.
* **`priority`** (string or number, OPTIONAL): For enqueue/update_priority (e.g., "high", "normal", 10).
* **`item_id`** (string, OPTIONAL/REQUIRED): Unique ID for the queued item/request. Generated by server on enqueue, used for subsequent actions.
* **`agent_id`** (string, OPTIONAL): For assign_agent/agent_accept/agent_reject, the user_id of the agent.
* **`details`** (object, OPTIONAL): For enqueue, any relevant information about the request (e.g., problem description, preferred language).
* **`status_info`** (object, OPTIONAL, usually in server response): For get_status, e.g., { "position": 5, "estimated_wait_time_seconds": 300 }.
* **`required_skills`** (array of strings, OPTIONAL): For enqueue, list of skills/tags required to handle this request (e.g., ["vpn", "network", "tier2"]).

**Example (Enqueue Customer)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "8a6f9e7b-5ba0-2daD-bd6d-5fc27b8e9f0e",
  "timestamp": "2025-06-01T01:40:00Z",
  "from": {
    "user_id": "u-customer-jane",
    "device_id": "d-customer-jane-web"
  },
  "to": { // To call center service/server
    "call_center_id": "cc-main-service"
  },
  "type": "QUEUE",
  "payload": {
    "action": "enqueue",
    "queue_id": "support_technical_tier1",
    "customer_id": "u-customer-jane",
    "priority": "high",
    "details": {
      "problem_summary": "Cannot connect to VPN",
      "product_version": "QSIP Client 2.5"
    },
    "required_skills": ["vpn", "network", "tier2"]
  }
}
```

### 5.17. CALENDAR

**Purpose**: Manages calendar events (meetings, appointments) related to QSIP users or groups.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED):
  * `"create_event"`
  * `"update_event"`
  * `"delete_event"`
  * `"get_event"`
  * `"list_events"` (e.g., for a user or group within a time range)
  * `"respond_to_invite"` (e.g., accept/decline a calendar event invite)
* **`event_id`** (string, OPTIONAL/REQUIRED): Unique ID for the calendar event.
* **`title`** (string, OPTIONAL): For create/update, title of the event.
* **`description`** (string, OPTIONAL): For create/update, description of the event.
* **`start_time`** (string, OPTIONAL): For create/update, ISO 8601 start time.
* **`end_time`** (string, OPTIONAL): For create/update, ISO 8601 end time.
* **`duration_minutes`** (number, OPTIONAL): For create/update, alternative to end_time.
* **`participants`** (array of objects, OPTIONAL): For create/update. Each object:
  * **`user_id`** (string, REQUIRED)
  * **`role`** (string, OPTIONAL, e.g., "organizer", "attendee", "optional_attendee")
  * **`response_status`** (string, OPTIONAL, e.g., "accepted", "declined", "tentative", "needs_action")
  * **`location`** (string, OPTIONAL): Physical location or QSIP meeting link.
  * **`recurrence_rule`** (string, OPTIONAL): iCalendar RRULE string for recurring events.
  * **`response`** (string, OPTIONAL): For respond_to_invite, the user's response (e.g., "accept", "decline", "tentative").
  * **`query_start_time`** (string, OPTIONAL): For list_events.
  * **`query_end_time`** (string, OPTIONAL): For list_events.

**Example (Create Calendar Event)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "9b7a0f8c-6cb1-3ebE-ce7e-6ad38c9fa01f",
  "timestamp": "2025-06-01T01:45:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To calendar service/server
  "type": "CALENDAR",
  "payload": {
    "action": "create_event",
    "event_id": "evt-weekly-qsip-sync-001",
    "title": "Weekly QSIP Sync Meeting",
    "description": "Discuss progress on QSIP development and upcoming features.",
    "start_time": "2025-06-07T15:00:00Z",
    "duration_minutes": 60,
    "participants": [
      { "user_id": "u-alice", "role": "organizer", "response_status": "accepted" },
      { "user_id": "u-bob", "role": "attendee", "response_status": "needs_action" },
      { "user_id": "u-charlie", "role": "attendee", "response_status": "needs_action" }
    ],
    "location": "QSIP Group: g-new-dev-team-alpha (auto-join link)"
  }
}
```

### 5.18. SCHEDULE

**Purpose**: Schedules a QSIP message (typically a MESSAGE type) to be sent or processed at a future time. This is more for server-side scheduling than the schedule_time hint in a MESSAGE.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED):
  * `"create_schedule"`
  * `"update_schedule"`
  * `"delete_schedule"`
  * `"get_schedule"`
* **`schedule_id`** (string, OPTIONAL/REQUIRED): Unique ID for the scheduled task.
* **`scheduled_message`** (object, REQUIRED for create): The full QSIP message object to be sent/processed. This message MUST NOT itself be a SCHEDULE message.
* **`schedule_time`** (string, REQUIRED for create): ISO 8601 timestamp for when the message should be processed.
* **`recurrence_rule`** (string, OPTIONAL): iCalendar RRULE for recurring scheduled messages.

**Example (Schedule a Reminder Message)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "ac8b1a9d-7dc2-4fcF-df8f-7be49da0b12a",
  "timestamp": "2025-06-01T01:50:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To scheduling service/server
  "type": "SCHEDULE",
  "payload": {
    "action": "create_schedule",
    "schedule_id": "sched-report-reminder-001",
    "schedule_time": "2025-06-03T17:00:00Z",
    "scheduled_message": {
      "qsip_version": "1.0",
      "message_id": "msg-to-be-scheduled-uuid", // Server will generate a new one on delivery
      "timestamp": "2025-06-03T17:00:00Z", // Will be updated on delivery
      "from": { // Server may act as sender or use original user_id
        "user_id": "u-system-reminder",
        "device_id": "d-server-scheduler"
      },
      "to": {
        "group_id": "g-new-dev-team-alpha"
      },
      "type": "MESSAGE",
      "payload": {
        "message_type": "text",
        "content": "Friendly reminder: Submit your weekly progress reports by EOD!"
      }
    }
  }
}
```

### 5.19. ANALYTICS

**Purpose**: Used by clients to report usage metrics or events to an analytics service, or by servers to push aggregated analytics.

**`session_id`**: Can be present if analytics are session-specific, otherwise null or omitted.

**Payload Schema**:

* **`event_name`** (string, REQUIRED): Name of the analytic event (e.g., "app_opened", "call_started", "message_sent", "feature_used").
* **`metric`** (string, OPTIONAL): Specific metric being reported if different from event_name (e.g. "active_users", "call_duration_seconds").
* **`value`** (any, OPTIONAL): Value associated with the metric (e.g., 350, 120.5, "success").
* **`dimensions`** (object, OPTIONAL): Key-value pairs providing context/dimensions for the event (e.g., { "platform": "Android", "app_version": "3.0" }).
* **`session_details`** (object, OPTIONAL): If related to a session, details like duration, participants.

**Example (Client reporting an event)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "bd9c2baE-8ed3-5ad0-ea90-8cf50eb1c23b",
  "timestamp": "2025-06-01T01:55:00Z",
  "from": {
    "user_id": "u-bob",
    "device_id": "d-bob-desktop"
  },
  "to": {}, // To analytics service/server
  "type": "ANALYTICS",
  "payload": {
    "event_name": "call_ended",
    "metric": "call_duration_seconds",
    "value": 753,
    "dimensions": {
      "call_type": "p2p_video",
      "network_type": "wifi"
    },
    "session_details": {
        "session_id": "s-call-alice-bob-123", // example session
        "peer_count": 2
    }
  }
}
```

### 5.20. MODERATION

**Purpose**: Allows authorized users (moderators, admins) to perform moderation actions within groups or on users.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The moderation action.
  * `"mute_user_in_group"`: Prevent a user from sending messages in a specific group.
  * `"unmute_user_in_group"`
  * `"kick_user_from_group"`: Remove a user from a group.
  * `"ban_user_from_group"`: Permanently ban a user from a group.
  * `"unban_user_from_group"`
  * `"delete_message"`: (Admin/Mod delete) Delete any user's message.
  * `"warn_user"`: Issue a warning to a user.
  * `"suspend_user_account"`: Globally suspend a user's account.
* **`target_user_id`** (string, REQUIRED for user-specific actions).
* **`target_group_id`** (string, OPTIONAL): The group context for the action.
* **`target_message_id`** (string, OPTIONAL): For delete_message.
* **`reason`** (string, OPTIONAL): Reason for the moderation action.
* **`duration_seconds`** (number, OPTIONAL): For temporary actions like mute/suspend, duration in seconds. If omitted, action is permanent or until undone.
* **`expires_at`** (string, OPTIONAL): Alternative to duration_seconds, an ISO 8601 timestamp.

**Example (Mute user in group)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "ce0d3cbF-9fe4-6be1-fb01-9da61fc2d34c",
  "timestamp": "2025-06-01T02:00:00Z",
  "from": { // Moderator/Admin
    "user_id": "u-admin-charlie",
    "device_id": "d-admin-charlie-controlpc"
  },
  "to": {}, // To moderation service/server
  "type": "MODERATION",
  "payload": {
    "action": "mute_user_in_group",
    "target_user_id": "u-spammy-sam",
    "target_group_id": "g-new-dev-team-alpha",
    "reason": "Repeatedly posting off-topic content.",
    "duration_seconds": 86400 // Mute for 24 hours
  }
}
```

### 5.21. AUDIT_EVENT

**Purpose**: For logging significant actions and events for compliance, security auditing, and administrative review. Typically generated by the server or privileged clients.

**`session_id`**: null or omitted, unless event is tied to a specific session.

**Payload Schema**:

* **`event_type`** (string, REQUIRED): Type of event being audited (e.g., "user_login", "user_logout", "password_changed", "admin_action", "file_downloaded", "group_created", "data_export").
* **`actor_user_id`** (string, OPTIONAL): The user_id who performed the action.
* **`actor_device_id`** (string, OPTIONAL): The device used by the actor.
* **`target_resource_type`** (string, OPTIONAL): Type of resource affected (e.g., "user", "group", "message", "file").
* **`target_resource_id`** (string, OPTIONAL): ID of the resource affected.
* **`ip_address`** (string, OPTIONAL): IP address associated with the event.
* **`user_agent`** (string, OPTIONAL): User agent string of the client.
* **`details`** (object, OPTIONAL): Any additional relevant information about the event.
* **`status`** (string, OPTIONAL, default: "success"): e.g., "success", "failure".
* **`failure_reason`** (string, OPTIONAL): If status is "failure".

**Example (User Login Event)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "df1e4dc0-a0f5-7cf2-ac12-aea72ad3e45d",
  "timestamp": "2025-06-01T02:05:00Z",
  "from": { // Usually server internal, or specific service
    "user_id": "u-system-audit",
    "device_id": "d-qsip-server-main"
  },
  "to": {}, // To audit log system
  "type": "AUDIT_EVENT",
  "payload": {
    "event_type": "user_login_success",
    "actor_user_id": "u-alice",
    "actor_device_id": "d-alice-mobile",
    "ip_address": "198.51.100.23",
    "user_agent": "qsip-mobile/3.0 (Android 13; Pixel 8)",
    "details": {
      "auth_method": "jwt_refresh_token"
    },
    "status": "success"
  }
}
```

### 5.22. ERROR

**Purpose**: Sent by an endpoint (client or server) to indicate that an error occurred while processing a request or that a QSIP-level error condition was met.

**`session_id`**: MAY be present if the error relates to a specific session.

**Payload Schema**:

* **`code`** (number, REQUIRED): A numeric error code. See Section 9.2 for code ranges.
* **`reason`** (string, REQUIRED): A short, machine-readable error reason phrase (e.g., "MalformedPayload", "Unauthorized", "NotFound", "SessionExpired").
* **`detail`** (string, OPTIONAL): A more detailed, human-readable explanation of the error.
* **`target_message_id`** (string, OPTIONAL): The message_id of the original message that caused this error, if applicable.

**Normative Requirements**:

* Endpoints MUST use this message type to report QSIP protocol errors.

**Example**:

```json
{
  "qsip_version": "1.0",
  "message_id": "e02f5ed1-b1a6-8df3-bd23-bfa83be4f56e",
  "timestamp": "2025-06-01T00:50:02Z",
  "from": { // The endpoint that detected the error
    "user_id": "u-qsip-server-main",
    "device_id": "d-server-instance-1"
  },
  "to": { // The originator of the problematic message
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "type": "ERROR",
  "session_id": "s-call-alice-bob-123", // If error is session-specific
  "payload": {
    "code": 4001,
    "reason": "MalformedPayload",
    "detail": "Missing required field 'media_types' in INVITE payload.",
    "target_message_id": "fdee98d1-fcbf-4def-b7d6-ebbd2d6718b0" // ID of the problematic INVITE
  }
}
```

### 5.23. KEY_EXCHANGE

**Purpose**: Used for exchanging cryptographic keys, primarily for end-to-end media encryption like SFrame.

**`session_id`**: REQUIRED, as key exchange is typically per-session.

**Payload Schema**:

* **`mechanism`** (string, REQUIRED, default: "sframe"): The E2EE mechanism these keys are for.
* **`sframe_params`** (object, OPTIONAL if mechanism is "sframe"): Parameters for SFrame.
* **`key_material`** (string, REQUIRED): Base64 encoded cryptographic key material.
* **`key_id`** (string or number, REQUIRED): Identifier for this key.
* **`cipher_suite`** (string, OPTIONAL): SFrame cipher suite (e.g., "AES_CM_128_HMAC_SHA256_80").
* **`epoch`** (number, OPTIONAL): Key epoch or generation number, for ratcheting.
* **`public_key`** (string, OPTIONAL): For asymmetric key exchange steps, a public key.
* **`signature`** (string, OPTIONAL): Signature to verify authenticity of exchanged keys.

**Normative Requirements**:

* Key exchange messages MUST be protected by the underlying QUIC/TLS transport encryption.
* Keys SHOULD be rotated periodically as recommended by the E2EE mechanism.

**Example (SFrame Key Parameters)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "f13a6fe2-c2b7-9ef4-ce34-cb094cf5a67f",
  "timestamp": "2025-06-01T00:50:06Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {
    "user_id": "u-bob"
  },
  "type": "KEY_EXCHANGE",
  "session_id": "s-call-alice-bob-123",
  "payload": {
    "mechanism": "sframe",
    "sframe_params": {
      "key_material": "b64EncodedSymmetricKeyOrSaltGoesHere...",
      "key_id": "1",
      "cipher_suite": "AES_GCM_128_SHA256", // Example if SFrame adopts explicit suites
      "epoch": 42
    }
  }
}
```

### 5.24. MEDIA_UPDATE

**Purpose**: Allows for mid-session modification of media streams, such as adding or removing a track, or simple codec changes, without a full re-INVITE. This is a lightweight alternative to full SDP renegotiation.

**`session_id`**: REQUIRED.

**Payload Schema**:

* **`add_tracks`** (array of objects, OPTIONAL): List of media tracks to add. Each object:
  * **`track_id`** (string, REQUIRED): A unique identifier for this new track.
  * **`kind`** (string, REQUIRED): Type of media (e.g., "audio", "video", "screenshare").
  * **`codec`** (string, REQUIRED): Codec string (e.g., "opus/48000/2", "av1/90000").
  * **`label`** (string, OPTIONAL): User-friendly label for the track.
  * **`direction`** (string, OPTIONAL, default: "sendrecv"): e.g., "sendonly", "recvonly".
* **`remove_tracks`** (array of strings, OPTIONAL): List of track_ids to remove.
* **`update_tracks`** (array of objects, OPTIONAL): List of media tracks to update (e.g. change direction). Each object similar to add_tracks but targets an existing track_id.

**Normative Requirements**:

* Used for incremental changes to media setup. For major changes (e.g., completely different set of media types), a new INVITE (re-INVITE) might be more appropriate.
* The recipient MUST acknowledge with a corresponding MEDIA_UPDATE (confirming changes) or an ERROR.

**Example (Add a video track mid-call)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "024b7af3-d3c8-0fa5-df45-dc1a5da6b78a",
  "timestamp": "2025-06-01T01:12:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {
    "user_id": "u-bob"
  },
  "type": "MEDIA_UPDATE",
  "session_id": "s-call-alice-bob-123",
  "payload": {
    "add_tracks": [
      {
        "track_id": "video-alice-2",
        "kind": "video",
        "codec": "av1/90000",
        "label": "Alice's Main Camera"
      }
    ],
    "remove_tracks": null // Or ["old-track-id"]
  }
}
```

### 5.25. USER_PROFILE

**Purpose**: Used to set or query user profile information, including custom status messages or other metadata not covered by standard PRESENCE.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED):
  * `"set"`: Set profile attributes for the sending user.
  * `"get"`: Get profile attributes for a target user.
* **`target_user_id`** (string, OPTIONAL): For get, the user_id whose profile is requested. If omitted in get, implies requesting own profile. For set, this is implicitly the from.user_id.
* **`profile_data`** (object, REQUIRED for set, OPTIONAL for get response): Key-value pairs of profile attributes. Examples:
  * **`display_name`** (string)
  * **`avatar_url`** (string)
  * **`custom_status_emoji`** (string)
  * **`custom_status_text`** (string)
  * **`custom_status_expires_at`** (string, ISO 8601)
  * **`pronouns`** (string)
  * **`department`** (string)
  * **`timezone`** (string, e.g., "America/New_York")

**Example (Set Custom Status)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "135c8ba4-e4d9-1ab6-ea56-ed2b6eb7c89b",
  "timestamp": "2025-06-01T09:00:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-desktop"
  },
  "to": {}, // To profile service / server
  "type": "USER_PROFILE",
  "payload": {
    "action": "set",
    "profile_data": {
      "custom_status_emoji": "🏖️",
      "custom_status_text": "On vacation until June 15th",
      "custom_status_expires_at": "2025-06-15T23:59:59Z",
      "timezone": "Europe/Paris"
    }
  }
}
```

### 5.26. SYNC

**Purpose**: Used for synchronization between devices or sessions.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The synchronization action.
  * `"request"`: Request synchronization.
  * `"response"`: Respond to a synchronization request.
* **`device_id`** (string, REQUIRED): The ID of the device requesting or responding to synchronization.
* **`timestamp`** (string, REQUIRED): ISO 8601 timestamp of the synchronization action.
* **`data`** (object, OPTIONAL): Any additional data related to the synchronization action.

**Example (Request Synchronization)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "146d9e8f-0a1b-2c3d-4e5f-6g7h8i9j0k1l",
  "timestamp": "2025-06-01T01:55:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To synchronization service/server
  "type": "SYNC",
  "payload": {
    "action": "request",
    "device_id": "d-bob-desktop",
    "timestamp": "2025-06-01T01:55:00Z"
  }
}
```

### 5.27. PUSH

**Purpose**: Used for push notifications to clients.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The push notification action.
  * `"subscribe"`: Subscribe to a topic.
  * `"unsubscribe"`: Unsubscribe from a topic.
  * `"send"`: Send a push notification.
* **`topic`** (string, REQUIRED): The topic to subscribe to or unsubscribe from.
* **`data`** (object, REQUIRED): The data to send in the push notification.

**Example (Subscribe to a Topic)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "157e0f9g-1b2c-3d4e-5f6g-7h8i9j0k1l2m",
  "timestamp": "2025-06-01T02:00:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To push notification service/server
  "type": "PUSH",
  "payload": {
    "action": "subscribe",
    "topic": "new-feature-updates",
    "data": {
      "title": "New Feature: Voice Messages",
      "message": "We've added voice messages to our app!"
    }
  }
}
```

### 5.28. SEARCH

**Purpose**: Used for searching for users, groups, or messages.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The search action.
  * `"user"`: Search for users.
  * `"group"`: Search for groups.
  * `"message"`: Search for messages.
* **`query`** (string, REQUIRED): The search query.
* **`limit`** (number, OPTIONAL): Maximum number of results to return.
* **`offset`** (number, OPTIONAL): Offset for paginated results.

**Example (Search for Users)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "168f1g0h-2c3d-4e5f-6g7h-8i9j0k1l2m3n",
  "timestamp": "2025-06-01T02:05:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To search service/server
  "type": "SEARCH",
  "payload": {
    "action": "user",
    "query": "John",
    "limit": 10,
    "offset": 0
  }
}
```

### 5.29. ROSTER

**Purpose**: Used for managing user contacts or "roster" in a messaging application.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The roster action.
  * `"add"`: Add a new contact.
  * `"remove"`: Remove an existing contact.
  * `"update"`: Update an existing contact's information.
* **`contact_id`** (string, REQUIRED): The ID of the contact to add, remove, or update.
* **`contact_info`** (object, REQUIRED): The information about the contact.

**Example (Add a Contact)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "179g2h1i-3d4e-5f6g-7h8i-9j0k1l2m3n4o5p",
  "timestamp": "2025-06-01T02:10:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To roster service/server
  "type": "ROSTER",
  "payload": {
    "action": "add",
    "contact_id": "c-john-doe",
    "contact_info": {
      "display_name": "John Doe",
      "status": "online",
      "last_seen": "2025-06-01T02:10:00Z"
    }
  }
}
```

### 5.30. CHANNEL

**Purpose**: Used for managing communication channels or "channels" in a messaging application.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The channel action.
  * `"create"`: Create a new channel.
  * `"delete"`: Delete an existing channel.
  * `"join"`: Join a channel.
  * `"leave"`: Leave a channel.
* **`channel_id`** (string, REQUIRED): The ID of the channel to create, delete, or join.
* **`name`** (string, OPTIONAL): The name of the channel.
* **`description`** (string, OPTIONAL): The description of the channel.

**Example (Create a Channel)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "180h3i2j-4e5f-6g7h-8i9j-0k1l2m3n4o5p6q",
  "timestamp": "2025-06-01T02:15:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To channel service/server
  "type": "CHANNEL",
  "payload": {
    "action": "create",
    "channel_id": "c-team-project",
    "name": "Team Project Chat",
    "description": "Discussing project updates and milestones."
  }
}
```

### 5.31. BOT

**Purpose**: Used for managing bots or automated agents in a messaging application.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The bot action.
  * `"add"`: Add a new bot.
  * `"remove"`: Remove an existing bot.
  * `"update"`: Update an existing bot's information.
* **`bot_id`** (string, REQUIRED): The ID of the bot to add, remove, or update.
* **`bot_info`** (object, REQUIRED): The information about the bot.

**Example (Add a Bot)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "191i4j3k-5f6g-7h8i-9j0k-1l2m3n4o5p6q7r",
  "timestamp": "2025-06-01T02:20:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To bot service/server
  "type": "BOT",
  "payload": {
    "action": "add",
    "bot_id": "b-project-assistant",
    "bot_info": {
      "name": "Project Assistant",
      "description": "Helps manage project tasks and communications."
    }
  }
}
```

### 5.32. VOICEMAIL

**Purpose**: Used for managing voice messages or voicemail in a messaging application.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The voicemail action.
  * `"add"`: Add a new voicemail.
  * `"remove"`: Remove an existing voicemail.
  * `"get"`: Get information about a voicemail.
* **`voicemail_id`** (string, REQUIRED): The ID of the voicemail to add, remove, or get.
* **`voicemail_info`** (object, REQUIRED): The information about the voicemail.

**Example (Add a Voicemail)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "1a2b3c4d-5e6f-7g8h-9i0j-klmnopqrstuv",
  "timestamp": "2025-06-01T02:25:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To voicemail service/server
  "type": "VOICEMAIL",
  "payload": {
    "action": "add",
    "voicemail_id": "v-important-meeting",
    "voicemail_info": {
      "subject": "Important Meeting Notes",
      "recording_url": "https://example.com/voicemail/important-meeting.mp3",
      "transcription": "Transcription of the voicemail recording..."
    }
  }
}
```

### 5.33. FEDERATION

**Purpose**: Used for managing federation or cross-platform communication between different QSIP servers.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The federation action.
  * `"add_server"`: Add a new server to the federation.
  * `"remove_server"`: Remove an existing server from the federation.
  * `"get_servers"`: Get information about all servers in the federation.
* **`server_id`** (string, REQUIRED): The ID of the server to add, remove, or get.
* **`server_info`** (object, REQUIRED): The information about the server.

**Example (Add a Server to Federation)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "1b2c3d4e-5f6g-7h8i-9j0k-1l2m3n4o5p6q7r8s",
  "timestamp": "2025-06-01T02:30:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To federation service/server
  "type": "FEDERATION",
  "payload": {
    "action": "add_server",
    "server_id": "s-new-server",
    "server_info": {
      "name": "New Server",
      "description": "A new server for our federation.",
      "url": "https://new-server.example.com"
    }
  }
}
```

### 5.34. WORKSPACE

**Purpose**: Used for managing workspaces or "workspaces" in a messaging application.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The workspace action.
  * `"create"`: Create a new workspace.
  * `"delete"`: Delete an existing workspace.
  * `"get"`: Get information about a workspace.
* **`workspace_id`** (string, REQUIRED): The ID of the workspace to create, delete, or get.
* **`workspace_info`** (object, REQUIRED): The information about the workspace.

**Example (Create a Workspace)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "1c2d3e4f-5g6h-7i8j-0k1l-2m3n4o5p6q7r8s9t",
  "timestamp": "2025-06-01T02:35:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To workspace service/server
  "type": "WORKSPACE",
  "payload": {
    "action": "create",
    "workspace_id": "w-team-project",
    "workspace_info": {
      "name": "Team Project Workspace",
      "description": "Collaboration space for the team project."
    }
  }
}
```

### 5.35. STORY

**Purpose**: Used for managing stories or "stories" in a messaging application.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The story action.
  * `"add"`: Add a new story.
  * `"remove"`: Remove an existing story.
  * `"get"`: Get information about a story.
* **`story_id`** (string, REQUIRED): The ID of the story to add, remove, or get.
* **`story_info`** (object, REQUIRED): The information about the story.

**Example (Add a Story)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "1d2e3f4g-5h6i-7j8k-0l1m-2n3o4p5q6r7s8t9u",
  "timestamp": "2025-06-01T02:40:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To story service/server
  "type": "STORY",
  "payload": {
    "action": "add",
    "story_id": "s-team-project-story-001",
    "story_info": {
      "title": "Team Project Milestone",
      "description": "Discussing progress on the team project.",
      "image_url": "https://example.com/story-image.jpg"
    }
  }
}
```

### 5.36. FORM

**Purpose**: Used for creating and managing forms or "forms" in a messaging application.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The form action.
  * `"create"`: Create a new form.
  * `"delete"`: Delete an existing form.
  * `"get"`: Get information about a form.
* **`form_id`** (string, REQUIRED): The ID of the form to create, delete, or get.
* **`form_info`** (object, REQUIRED): The information about the form.

**Example (Create a Form)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "1e2f3g4h-5i6j-7k8l-0m1n-2o3p4q5r6s7t8u9v",
  "timestamp": "2025-06-01T02:45:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To form service/server
  "type": "FORM",
  "payload": {
    "action": "create",
    "form_id": "f-feedback-survey",
    "form_info": {
      "title": "Feedback Survey",
      "description": "Please provide your feedback on our services.",
      "questions": [
        { "id": "q1", "type": "text", "label": "How satisfied are you with our services?" },
        { "id": "q2", "type": "textarea", "label": "What improvements would you like to see?" },
        { "id": "q3", "type": "radio", "label": "How often do you use our services?", "options": ["Daily", "Weekly", "Monthly"] }
      ]
    }
  }
}
```

### 5.37. PAYMENT

**Purpose**: Used for managing payments or "payments" in a messaging application.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The payment action.
  * `"add"`: Add a new payment method.
  * `"remove"`: Remove an existing payment method.
  * `"get"`: Get information about a payment method.
* **`payment_id`** (string, REQUIRED): The ID of the payment method to add, remove, or get.
* **`payment_info`** (object, REQUIRED): The information about the payment method.

**Example (Add a Payment Method)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "1f2g3h4i-5j6k-7l8m-0n1o-2p3q4r5s6t7u8v",
  "timestamp": "2025-06-01T02:50:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To payment service/server
  "type": "PAYMENT",
  "payload": {
    "action": "add",
    "payment_id": "p-credit-card",
    "payment_info": {
      "card_number": "****-****-****-1234",
      "expiration_date": "12/25",
      "cvv": "123"
    }
  }
}
```

### 5.38. TRANSCRIPT

**Purpose**: Used for managing transcripts or "transcripts" in a messaging application.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The transcript action.
  * `"add"`: Add a new transcript.
  * `"remove"`: Remove an existing transcript.
  * `"get"`: Get information about a transcript.
* **`transcript_id`** (string, REQUIRED): The ID of the transcript to add, remove, or get.
* **`transcript_info`** (object, REQUIRED): The information about the transcript.

**Example (Add a Transcript)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "1g2h3i4j-5k6l-7m8n-0o1p-2q3r4s5t6u7v8w",
  "timestamp": "2025-06-01T02:55:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To transcript service/server
  "type": "TRANSCRIPT",
  "payload": {
    "action": "add",
    "transcript_id": "t-important-meeting",
    "transcript_info": {
      "title": "Important Meeting Transcript",
      "description": "Notes from the recent important meeting.",
      "recording_url": "https://example.com/transcript/important-meeting.mp3"
    }
  }
}
```

### 5.39. BACKUP

**Purpose**: Used for managing backups or "backups" in a messaging application.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The backup action.
  * `"add"`: Add a new backup.
  * `"remove"`: Remove an existing backup.
  * `"get"`: Get information about a backup.
* **`backup_id`** (string, REQUIRED): The ID of the backup to add, remove, or get.
* **`backup_info`** (object, REQUIRED): The information about the backup.

**Example (Add a Backup)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "1h2i3j4k-5l6m-7n8o-0p1q-2r3s4t5u6v7w8x",
  "timestamp": "2025-06-01T03:00:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To backup service/server
  "type": "BACKUP",
  "payload": {
    "action": "add",
    "backup_id": "b-important-data",
    "backup_info": {
      "description": "Important data backup",
      "file_url": "https://example.com/backup/important-data.zip"
    }
  }
}
```

### 5.40. MFA

**Purpose**: Used for managing multi-factor authentication (MFA) or "MFA" in a messaging application.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The MFA action.
  * `"add"`: Add a new MFA method.
  * `"remove"`: Remove an existing MFA method.
  * `"get"`: Get information about an MFA method.
* **`mfa_id`** (string, REQUIRED): The ID of the MFA method to add, remove, or get.
* **`mfa_info`** (object, REQUIRED): The information about the MFA method.

**Example (Add an MFA Method)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "1i2j3k4l-5m6n-7o8p-0q1r-2s3t4u5v6w7x8y",
  "timestamp": "2025-06-01T03:05:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To MFA service/server
  "type": "MFA",
  "payload": {
    "action": "add",
    "mfa_id": "m-security-question",
    "mfa_info": {
      "question": "What is your mother's maiden name?",
      "answer": "Smith"
    }
  }
}
```

### 5.41. SUBSCRIPTION

**Purpose**: Used for managing content subscriptions to threads, channels, automated tools, or event-based notifications.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The subscription action.
  * `"subscribe"`: Subscribe to content or events.
  * `"unsubscribe"`: Unsubscribe from content or events.
  * `"update"`: Update subscription preferences.
  * `"list"`: List current subscriptions.
* **`subscription_type`** (string, REQUIRED): Type of subscription.
  * `"thread"`: Subscribe to thread updates.
  * `"channel"`: Subscribe to channel activity.
  * `"user"`: Subscribe to user activity.
  * `"bot"`: Subscribe to bot/tool outputs.
  * `"event"`: Subscribe to specific events.
  * `"summary"`: Subscribe to periodic summaries.
* **`target_id`** (string, REQUIRED for subscribe/unsubscribe): ID of the resource to subscribe to.
* **`filters`** (object, OPTIONAL): Filtering criteria for the subscription.
  * **`keywords`** (array of strings, OPTIONAL): Only notify for content containing these keywords.
  * **`users`** (array of strings, OPTIONAL): Only notify for activity from these users.
  * **`event_types`** (array of strings, OPTIONAL): Specific events to subscribe to.
* **`delivery`** (object, OPTIONAL): Delivery preferences.
  * **`frequency`** (string, OPTIONAL): For summaries (e.g., "hourly", "daily", "weekly").
  * **`time`** (string, OPTIONAL): Preferred delivery time (ISO 8601 time).
  * **`format`** (string, OPTIONAL): Delivery format (e.g., "full", "summary", "digest").
  * **`max_items`** (number, OPTIONAL): Maximum items per delivery.

**Example (Subscribe to Thread Summary)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "1j2k3l4m-5n6o-7p8q-0r1s-2t3u4v5w6x7y",
  "timestamp": "2025-06-01T03:10:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {}, // To subscription service/server
  "type": "SUBSCRIPTION",
  "payload": {
    "action": "subscribe",
    "subscription_type": "summary",
    "target_id": "thread-project-discussion-123",
    "filters": {
      "keywords": ["milestone", "deadline", "decision"],
      "users": ["u-manager-bob", "u-lead-charlie"]
    },
    "delivery": {
      "frequency": "daily",
      "time": "09:00:00",
      "format": "digest",
      "max_items": 10
    }
  }
}
```

**Example (Subscribe to Bot Output)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "2k3l4m5n-6o7p-8q9r-0s1t-2u3v4w5x6y7z",
  "timestamp": "2025-06-01T03:15:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {},
  "type": "SUBSCRIPTION",
  "payload": {
    "action": "subscribe",
    "subscription_type": "bot",
    "target_id": "bot-analytics-reporter",
    "filters": {
      "event_types": ["daily_report", "anomaly_detected"]
    },
    "delivery": {
      "format": "full"
    }
  }
}
```

### 5.42. INTERACTIVE

**Purpose**: Handles user interactions with rich content elements like card buttons, form submissions, canvas actions, etc.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`interaction_type`** (string, REQUIRED): Type of interaction.
  * `"button_click"`: User clicked a button in a card/carousel.
  * `"form_submit"`: User submitted a form.
  * `"canvas_action"`: Action on a collaborative canvas.
  * `"embed_event"`: Event from an embedded app/widget.
* **`source_message_id`** (string, REQUIRED): The message_id containing the interactive element.
* **`action_id`** (string, REQUIRED): The action_id from the interactive element.
* **`payload`** (object, OPTIONAL): Data associated with the interaction.
  * For button clicks: The payload from the button definition.
  * For form submissions: Form field values.
  * For canvas actions: Drawing/editing data.
* **`response_type`** (string, OPTIONAL): How to handle the response.
  * `"ephemeral"`: Response visible only to the interacting user.
  * `"in_channel"`: Response visible to all in the channel/chat.
  * `"update"`: Update the original message.
  * `"new_message"`: Send as a new message.

**Example (Button Click Interaction)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "2m3n4o5p-6q7r-8s9t-0u1v-2w3x4y5z6a7b",
  "timestamp": "2025-06-01T03:20:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {
    "user_id": "u-bot-assistant"
  },
  "type": "INTERACTIVE",
  "payload": {
    "interaction_type": "button_click",
    "source_message_id": "2a9f3d1b-95e4-4c3c-bd8c-9e6b1a2d3c4d",
    "action_id": "join-meeting-123",
    "payload": {
      "meeting_id": "standup-2025-06-01"
    },
    "response_type": "ephemeral"
  }
}
```

### 5.43. COMMAND

**Purpose**: Handles slash commands for quick actions and bot interactions.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`command_type`** (string, REQUIRED): Type of command.
  * `"slash"`: Slash command (e.g., /remind, /call).
  * `"menu"`: Context menu command.
  * `"shortcut"`: Global or message shortcut.
* **`command`** (string, REQUIRED): The command identifier (e.g., "remind", "status", "giphy").
* **`arguments`** (array of strings, OPTIONAL): Command arguments parsed from user input.
* **`raw_text`** (string, OPTIONAL): The full raw text after the command.
* **`context`** (object, OPTIONAL): Context where command was invoked.
  * **`channel_id`** (string, OPTIONAL): Channel where command was used.
  * **`message_id`** (string, OPTIONAL): Message ID for message shortcuts.
  * **`user_ids`** (array of strings, OPTIONAL): Selected users for user shortcuts.
* **`response_url`** (string, OPTIONAL): URL to send delayed responses to.
* **`trigger_id`** (string, OPTIONAL): ID for opening modals in response.

**Example (Slash Command)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "3n4o5p6q-7r8s-9t0u-1v2w-3x4y5z6a7b",
  "timestamp": "2025-06-01T03:25:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {
    "user_id": "u-bot-reminder"
  },
  "type": "COMMAND",
  "payload": {
    "command_type": "slash",
    "command": "remind",
    "arguments": ["me", "in", "2h", "to"],
    "raw_text": "me in 2h to review the PR",
    "context": {
      "channel_id": "g-dev-team"
    },
    "trigger_id": "trigger-123-abc"
  }
}
```

### 5.44. WEBHOOK

**Purpose**: Manages webhook configurations and webhook event delivery for external integrations.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`webhook_type`** (string, REQUIRED): Type of webhook.
  * `"incoming"`: External service sending data to QSIP.
  * `"outgoing"`: QSIP sending events to external service.
  * `"configure"`: Configure webhook settings.
* **`action`** (string, REQUIRED for configure): Configuration action.
  * `"create"`: Create new webhook.
  * `"update"`: Update webhook configuration.
  * `"delete"`: Remove webhook.
  * `"list"`: List configured webhooks.
* **`webhook_id`** (string, OPTIONAL): Unique identifier for the webhook.
* **`url`** (string, OPTIONAL): Webhook endpoint URL.
* **`events`** (array of strings, OPTIONAL): Events to subscribe to for outgoing webhooks.
  * Examples: `["message.created", "user.joined", "file.shared"]`
* **`secret`** (string, OPTIONAL): Shared secret for webhook signature verification.
* **`headers`** (object, OPTIONAL): Custom headers to include in webhook requests.
* **`data`** (object, OPTIONAL): Payload data for incoming webhooks.
* **`channel_id`** (string, OPTIONAL): Default channel for incoming webhook messages.

**Example (Incoming Webhook Data)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "4o5p6q7r-8s9t-0u1v-2w3x4y5z6a7b8c",
  "timestamp": "2025-06-01T03:30:00Z",
  "from": {
    "user_id": "u-webhook-github",
    "device_id": "d-webhook-service"
  },
  "to": {
    "channel_id": "g-dev-notifications"
  },
  "type": "WEBHOOK",
  "payload": {
    "webhook_type": "incoming",
    "webhook_id": "wh-github-123",
    "data": {
      "event": "push",
      "repository": "myorg/myrepo",
      "branch": "main",
      "commits": [
        {
          "id": "abc123",
          "message": "Fix: Resolved issue with webhooks",
          "author": "developer@example.com"
        }
      ]
    }
  }
}
```

### 5.45. WORKFLOW

**Purpose**: Manages automated workflows that execute multi-step processes based on triggers and conditions.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The workflow action.
  * `"create"`: Create a new workflow.
  * `"update"`: Update an existing workflow.
  * `"delete"`: Delete a workflow.
  * `"execute"`: Manually trigger a workflow.
  * `"pause"`: Pause a running workflow.
  * `"resume"`: Resume a paused workflow.
  * `"get_status"`: Get workflow execution status.
* **`workflow_id`** (string, REQUIRED for all except create): Unique workflow identifier.
* **`name`** (string, OPTIONAL): Human-readable workflow name.
* **`trigger`** (object, REQUIRED for create): What initiates the workflow.
  * **`type`** (string, REQUIRED): Trigger type (e.g., "schedule", "event", "webhook", "manual").
  * **`config`** (object, OPTIONAL): Trigger-specific configuration.
* **`steps`** (array of objects, REQUIRED for create): Workflow steps to execute.
  * **`id`** (string, REQUIRED): Step identifier.
  * **`type`** (string, REQUIRED): Step type (e.g., "send_message", "wait", "condition", "http_request").
  * **`config`** (object, REQUIRED): Step-specific configuration.
  * **`next`** (string or object, OPTIONAL): Next step(s) based on conditions.
* **`variables`** (object, OPTIONAL): Workflow variables and their initial values.
* **`execution_id`** (string, OPTIONAL): For status/control of specific execution.

**Example (Create Approval Workflow)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "5p6q7r8s-9t0u-1v2w-3x4y-5z6a7b8c9d0e",
  "timestamp": "2025-06-01T03:35:00Z",
  "from": {
    "user_id": "u-admin-alice",
    "device_id": "d-alice-desktop"
  },
  "to": {},
  "type": "WORKFLOW",
  "payload": {
    "action": "create",
    "workflow_id": "wf-expense-approval",
    "name": "Expense Approval Workflow",
    "trigger": {
      "type": "event",
      "config": {
        "event": "form.submitted",
        "form_id": "expense-request"
      }
    },
    "steps": [
      {
        "id": "notify_manager",
        "type": "send_message",
        "config": {
          "to": "{{submitter.manager}}",
          "template": "expense_approval_request",
          "include_form_data": true
        },
        "next": "wait_approval"
      },
      {
        "id": "wait_approval",
        "type": "wait",
        "config": {
          "for": "user_action",
          "timeout": "48h"
        },
        "next": {
          "approved": "notify_approved",
          "rejected": "notify_rejected",
          "timeout": "escalate"
        }
      }
    ]
  }
}
```

### 5.46. MODAL

**Purpose**: Displays interactive modal dialogs for forms, confirmations, and complex interactions.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The modal action.
  * `"open"`: Open a new modal.
  * `"update"`: Update an existing modal.
  * `"close"`: Close a modal.
  * `"push"`: Push a new view onto the modal stack.
  * `"submit"`: Submit modal form data.
* **`trigger_id`** (string, REQUIRED for open): Temporary ID from interaction that triggered the modal.
* **`view_id`** (string, OPTIONAL): Unique identifier for the modal view.
* **`view`** (object, REQUIRED for open/update): Modal view definition.
  * **`type`** (string, REQUIRED): View type (e.g., "modal", "home").
  * **`title`** (string, REQUIRED): Modal title.
  * **`blocks`** (array of objects, REQUIRED): UI blocks to display.
  * **`submit`** (object, OPTIONAL): Submit button configuration.
  * **`cancel`** (object, OPTIONAL): Cancel button configuration.
  * **`close`** (object, OPTIONAL): Close button configuration.
  * **`private_metadata`** (string, OPTIONAL): Hidden data passed to handlers.
* **`values`** (object, OPTIONAL for submit): Form values from modal submission.
* **`response_action`** (string, OPTIONAL): How to respond (e.g., "clear", "update", "push").

**Example (Open Form Modal)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "6q7r8s9t-0u1v-2w3x-4y5z-6a7b8c9d0e1f",
  "timestamp": "2025-06-01T03:40:00Z",
  "from": {
    "user_id": "u-bot-hr",
    "device_id": "d-bot-server"
  },
  "to": {
    "user_id": "u-alice"
  },
  "type": "MODAL",
  "payload": {
    "action": "open",
    "trigger_id": "trigger-456-def",
    "view": {
      "type": "modal",
      "title": "Request Time Off",
      "view_id": "modal-time-off-request",
      "blocks": [
        {
          "type": "input",
          "block_id": "start_date",
          "label": "Start Date",
          "element": {
            "type": "datepicker",
            "action_id": "start_date_selected"
          }
        },
        {
          "type": "input",
          "block_id": "end_date",
          "label": "End Date",
          "element": {
            "type": "datepicker",
            "action_id": "end_date_selected"
          }
        },
        {
          "type": "input",
          "block_id": "reason",
          "label": "Reason",
          "element": {
            "type": "textarea",
            "action_id": "reason_input",
            "placeholder": "Optional: Reason for time off"
          }
        }
      ],
      "submit": {
        "type": "button",
        "text": "Submit Request"
      },
      "close": {
        "type": "button",
        "text": "Cancel"
      }
    }
  }
}
```

### 5.47. CLIP

**Purpose**: Manages short video messages (clips) for asynchronous video communication.

**`session_id`**: null or omitted.

**Payload Schema**:

* **`action`** (string, REQUIRED): The clip action.
  * `"record"`: Start recording a clip.
  * `"upload"`: Upload a recorded clip.
  * `"share"`: Share clip to channel/user.
  * `"delete"`: Delete a clip.
  * `"transcribe"`: Request transcription.
* **`clip_id`** (string, REQUIRED for all except record): Unique clip identifier.
* **`title`** (string, OPTIONAL): Clip title.
* **`duration_ms`** (number, OPTIONAL): Clip duration in milliseconds.
* **`recording_data`** (object, OPTIONAL for upload): Recording information.
  * **`format`** (string, REQUIRED): Video format (e.g., "webm", "mp4").
  * **`resolution`** (string, OPTIONAL): Video resolution (e.g., "1280x720").
  * **`file_size`** (number, REQUIRED): Size in bytes.
  * **`url`** (string, REQUIRED): URL to uploaded video file.
  * **`thumbnail_url`** (string, OPTIONAL): URL to video thumbnail.
* **`share_targets`** (array of objects, OPTIONAL for share): Where to share the clip.
  * **`type`** (string, REQUIRED): Target type ("user", "channel", "group").
  * **`id`** (string, REQUIRED): Target ID.
* **`transcription`** (object, OPTIONAL): Transcription data.
  * **`text`** (string): Full transcript text.
  * **`language`** (string): Language code (e.g., "en-US").
  * **`timestamps`** (array): Word-level timestamps.

**Example (Share Recorded Clip)**:

```json
{
  "qsip_version": "1.0",
  "message_id": "7r8s9t0u-1v2w-3x4y-5z6a-7b8c9d0e1f2g",
  "timestamp": "2025-06-01T03:45:00Z",
  "from": {
    "user_id": "u-alice",
    "device_id": "d-alice-mobile"
  },
  "to": {},
  "type": "CLIP",
  "payload": {
    "action": "share",
    "clip_id": "clip-project-update-123",
    "title": "Quick Project Update",
    "duration_ms": 45000,
    "recording_data": {
      "format": "webm",
      "resolution": "1280x720",
      "file_size": 5242880,
      "url": "https://cdn.example.com/clips/project-update-123.webm",
      "thumbnail_url": "https://cdn.example.com/clips/project-update-123-thumb.jpg"
    },
    "share_targets": [
      {
        "type": "channel",
        "id": "g-dev-team"
      }
    ],
    "transcription": {
      "text": "Hey team, quick update on the project. We've completed the API integration...",
      "language": "en-US"
    }
  }
}
```

## 6. Session Management

### 6.1. Session Lifecycle

A QSIP session typically refers to a real-time communication exchange, like a call or conference, identified by a unique session_id. The lifecycle generally follows:

* **Initiation**: An endpoint sends an INVITE message with a new session_id.
* **Negotiation**: The recipient(s) respond with ACCEPT (including their media parameters) or REJECT. ICE negotiation may occur via candidates in INVITE/ACCEPT or subsequent messages. Key exchange (KEY_EXCHANGE) for E2EE also happens during this phase.
* **Active**: If accepted, the session becomes active. Media flows, and participants can send SESSION control messages, CHANNEL_UPDATEs, or MEDIA_UPDATEs.
* **Termination**: Any participant or the server can end the session by sending an END message.

### 6.2. State Machines

Detailed state machines for sessions, calls, queues, and other stateful interactions are essential for consistent implementation. These will be provided in Appendix B. A simplified session state machine:

| Current State | Event Received/Sent | Next State | Actions |
|---------------|-------------------|------------|---------|
| IDLE | Send INVITE | OUTGOING | Start INVITE timer |
| IDLE | Receive INVITE | INCOMING | Notify user, start ringing timer |
| OUTGOING | Receive ACCEPT | ACTIVE | Stop INVITE timer, establish media |
| OUTGOING | Receive REJECT | TERMINATED | Stop INVITE timer, notify user |
| OUTGOING | INVITE Timer Expires | TERMINATED | Notify user (timeout) |
| INCOMING | Send ACCEPT | ACTIVE | Stop ringing timer, establish media |
| INCOMING | Send REJECT | TERMINATED | Stop ringing timer, notify caller |
| INCOMING | Ringing Timer Expires | TERMINATED | Auto-reject (missed call) |
| ACTIVE | Send/Receive END | TERMINATED | Tear down media, notify user |
| ACTIVE | Send/Receive SESSION (hold) | ON_HOLD | Update UI, potentially pause media |
| ON_HOLD | Send/Receive SESSION (resume) | ACTIVE | Update UI, potentially resume media |
| (Any) | Fatal Error | TERMINATED | Log error, notify user |

## 7. Media Transport

QSIP signaling orchestrates media sessions, but the media itself is transported separately, typically leveraging QUIC datagrams or other UDP-based mechanisms.

### 7.1. Media Encapsulation

When using QUIC datagrams [RFC9221], media packets (e.g., RTP [RFC3550] or SFrame packets) SHOULD be sent directly within the datagrams.

Each datagram SHOULD ideally contain one media packet to minimize latency and simplify processing.

The use of RTP-over-QUIC, as described in [draft-ietf-avtcore-rtp-over-quic], is RECOMMENDED for structuring media transport, including header compression and multiplexing if needed. Standard RTP clock rates (e.g., 90000 Hz for video, 48000 Hz for Opus audio) SHOULD be used.

### 7.2. Codec Negotiation

Codec negotiation occurs primarily during session setup via the codecs field in INVITE and ACCEPT messages.

The offer/answer model is used: the initiator lists supported codecs, and the acceptor selects a common subset.

MEDIA_UPDATE messages MAY be used for lightweight codec adjustments or adding/removing tracks with specific codecs mid-session.

AV1 for video and Opus for audio are RECOMMENDED as default high-quality, royalty-free codecs. Support for other codecs like H.264 (for compatibility) MAY be included.

### 7.3. SFrame End-to-End Encryption

For end-to-end encrypted media, SFrame [draft-ietf-sframe] is RECOMMENDED.

Cryptographic keys for SFrame MUST be exchanged using KEY_EXCHANGE messages over the secure QSIP signaling channel.

The sframe_params in the KEY_EXCHANGE payload carry the necessary keying material, key IDs, and potentially epoch information for key ratcheting.

Implementations MUST follow SFrame specifications for packet protection and key management.

### 7.4. Simulcast and Scalable Video Coding (SVC)

To support varying network conditions and receiver capabilities in group calls, QSIP implementations SHOULD support simulcast (sending multiple resolutions/bitrates of the same video source) and/or SVC (a single video stream with multiple scalable layers).

The capabilities field in REGISTER and codec descriptions in INVITE/ACCEPT MAY indicate support for simulcast or SVC.

Specific parameters for simulcast layers or SVC layers MAY be signaled via extensions to the codecs object or within MEDIA_UPDATE messages. This area may require further specification or reference to existing WebRTC practices (e.g., RID/MID extensions in SDP).

### 7.5. NAT Traversal

QSIP leverages QUIC's properties (single port, connection migration) which can simplify NAT traversal compared to multi-port TCP/UDP protocols.

However, for P2P media, Interactive Connectivity Establishment (ICE) [RFC8445] is still RECOMMENDED.

ICE candidates SHOULD be exchanged in INVITE and ACCEPT messages, or via subsequent dedicated messages if needed. The ice_candidates array in these messages serves this purpose.

Support for STUN [RFC8489] and TURN [RFC8656] (both for signaling and media relays if QUIC proxying is used) is RECOMMENDED for robust connectivity. A relay=true flag or similar indication in ICE candidate JSON MAY signal a TURN candidate.

The QSIP server can act as an ICE negotiation orchestrator or a TURN relay itself.

## 8. Authentication and Authorization

### 8.1. Device Registration

Clients MUST register with a QSIP server using the REGISTER message (Section 5.1) after establishing a QUIC connection. This step is crucial for authenticating the user/device and establishing its presence.

### 8.2. Token-based Authentication

The auth_token field in the REGISTER message payload is the primary means of authentication.

JSON Web Tokens (JWT) [RFC7519] are RECOMMENDED for auth_token due to their widespread support and ability to carry claims.

The method of obtaining the initial auth_token (e.g., OAuth 2.0 flow, login with credentials to an IdP) is outside the scope of QSIP but MUST be secure.

### 8.3. Token Lifecycle

auth_tokens MUST have an expiration time. Clients SHOULD be prepared to refresh tokens.

Servers MAY provide a new token (e.g., in a RECEIPT to REGISTER or a dedicated AUTH_REFRESH_SUCCESS message) or expect clients to use a refresh token via an out-of-band mechanism.

If a token expires or is invalid, the server MUST reject QSIP messages requiring authentication with an appropriate ERROR (e.g., code 4011 "Unauthorized" or 4012 "TokenExpired").

Servers MUST provide a mechanism for token revocation in case of compromised devices or user logout.

## 9. Error Handling

### 9.1. ERROR Message Type

QSIP uses a dedicated ERROR message (Section 5.22) to report issues. This message includes a numeric code, a reason phrase, and optional details.

### 9.2. Error Codes

Error codes are grouped by range:

* **3xxx** - Request-Specific Success with Caveats (Rarely used, for future extension if needed)

* **4xxx** - Client-Side Errors: Problems with the client's request.
  * **4000-4009**: General Client Errors
    * **4001**: MalformedPayload (e.g., invalid JSON, missing required field)
    * **4002**: InvalidMessageType
    * **4003**: UnsupportedQsipVersion
    * **4004**: MessageTooLarge
  * **4010-4019**: Authentication/Authorization Errors
    * **4011**: Unauthorized (general authentication failure)
    * **4012**: TokenExpired
    * **4013**: InvalidToken
    * **4014**: InsufficientPermissions (user authenticated but not authorized for the action)
    * **4015**: RegistrationRequired
  * **4030-4039**: Request Forbidden/Denied
    * **4031**: Forbidden (generic denial)
    * **4032**: UserNotOnRoster / NotFriends
    * **4033**: TargetUserBlocked
  * **4040-4049**: Resource Not Found
    * **4041**: UserNotFound
    * **4042**: GroupNotFound
    * **4043**: SessionNotFound
    * **4044**: MessageNotFound
    * **4045**: ResourceNotFound (generic)
  * **4050-4059**: Method/Action Not Allowed
    * **4051**: InvalidActionForState (e.g., trying to ACCEPT an already active session)
    * **4052**: ActionNotSupported
  * **4060-4069**: Media/Capability Issues
    * **4061**: NoCommonCodec
    * **4062**: UnsupportedMediaType
    * **4063**: SdpNegotiationFailed
  * **4080-4089**: Timeouts
    * **4081**: RequestTimeout (general request timeout)
  * **4290-4299**: Rate Limiting
    * **4291**: TooManyRequests / RateLimitExceeded

* **5xxx** - Server-Side Errors: Problems on the server prevented fulfilling a valid request.
  * **5001**: InternalServerError
  * **5002**: ServiceUnavailable (e.g., dependent service down)
  * **5003**: DatabaseError
  * **5004**: QueueFull

* **6xxx** - Session/Media Specific Errors (Reported during an active session)
  * **6001**: MediaConnectionFailed
  * **6002**: KeyExchangeFailed

This list is not exhaustive. See IANA Considerations (Section 14.2) for registration of new error codes.

## 10. Extensibility

### 10.1. Custom Message Types

While this specification defines a core set of message types, QSIP is designed to be extensible. New message types MAY be defined. Organizations defining custom types SHOULD prefix them with a reverse domain name string (e.g., "com.example.custom_type") to avoid collisions, unless registered via an IANA process.

### 10.2. Payload Extensions

Custom fields MAY be added to the payload of standard or custom message types. Clients and servers MUST ignore unknown fields within a payload to ensure forward compatibility.

## 11. Performance Considerations

### 11.1. Latency

QSIP aims for low latency by:

Using QUIC's 0-RTT or 1-RTT connection establishment.
Multiplexing all communication over a single connection, avoiding head-of-line blocking between streams.
Using compact JSON payloads and QUIC datagrams for media.

### 11.2. Bandwidth Efficiency

JSON payloads are relatively verbose compared to binary formats. Implementations MAY use compression (e.g., DEFLATE, Brotli) at the QUIC stream level if negotiated, or a more compact serialization like CBOR if universally adopted in a future QSIP version or profile. For now, JSON is mandated for interoperability.
Efficient codecs (Opus, AV1) are recommended for media.
QUIC header and transport mechanisms are more efficient than TCP/TLS/HTTP stacks.

### 11.3. Congestion Control

QSIP relies on QUIC's built-in congestion control mechanisms (e.g., NewReno, CUBIC, BBR) to adapt to network conditions. Application-level rate limiting or prioritization (e.g., for critical signaling messages over large file transfer notifications) MAY be implemented by QSIP servers. DSCP marking recommendations MAY be provided in future revisions for QoS.

## 12. Security Considerations

Security is a fundamental design principle of QSIP.

* **Transport Security**: All QSIP communication MUST be over QUIC, which mandates TLS 1.3 encryption. This protects against eavesdropping, tampering, and replay of signaling messages at the transport layer.

* **Authentication**: Strong authentication of users and devices is enforced via the REGISTER message and auth_token. Implementations MUST use secure methods for token generation, management, and validation.

* **End-to-End Encryption (E2EE)**: For media, SFrame is RECOMMENDED for E2EE. KEY_EXCHANGE messages facilitate secure transfer of SFrame keys over the already encrypted QSIP channel. For messaging, a similar E2EE scheme (like Signal Protocol, OLM/Megolm) MAY be implemented by encrypting the content of MESSAGE payloads, with keys exchanged via KEY_EXCHANGE or a similar mechanism.

* **Authorization**: Servers MUST enforce authorization checks to ensure users can only perform actions and access data they are permitted to (e.g., joining private groups, moderating, accessing user profiles).

* **Data Validation**: All incoming QSIP messages and payloads MUST be rigorously validated to prevent injection attacks, denial of service, or crashes due to malformed data. This includes checking data types, lengths, and allowed values.

* **Denial of Service (DoS) Mitigation**: Servers SHOULD implement rate limiting, connection limits, and potentially QUIC-level mitigations (e.g., address validation) to protect against DoS attacks.

* **Replay Protection**: QUIC/TLS provides replay protection at the transport layer. For application-level idempotency where needed (e.g., financial transactions if QSIP were extended), message_id can be used.

* **Privacy**: See Section 13.

* **Message Integrity**: TLS ensures the integrity of messages in transit.

Implementers SHOULD consult standard security best practices for RTC applications and QUIC deployments.

## 13. Privacy Considerations

* **PII Minimization**: QSIP messages SHOULD only carry Personally Identifiable Information (PII) necessary for their function. For example, display_name in the from field is optional.

* **Data Encryption**: All data is encrypted in transit by QUIC/TLS. E2EE for media (SFrame) and messaging content provides additional privacy against server-side eavesdropping.

* **Logging**: Servers and clients SHOULD be configurable regarding the level of logging. Logs containing PII or sensitive message content MUST be protected with appropriate access controls and SHOULD be anonymized or pseudonymized where possible. Log retention policies SHOULD be defined.

* **User Consent**: Applications using QSIP MUST obtain user consent for collecting and processing PII, in accordance with applicable privacy regulations (e.g., GDPR, CCPA).

* **Presence and Profile Visibility**: Users SHOULD have granular control over who can see their presence information and detailed profile data.

* **Analytics**: If analytics data is collected, it SHOULD be anonymized or aggregated to protect individual user privacy. Explicit user consent for analytics data collection might be required.

* **Location Information**: If location data is shared (e.g., via MESSAGE type location), it MUST be with explicit user consent per instance or per application setting.

## 14. IANA Considerations

This document requests the creation of several IANA registries under a new "QUIC Signaling and Interaction Protocol (QSIP) Parameters" heading. The registration policy for new entries in these registries, unless otherwise specified, is "Specification Required" [RFC8126].

### 14.1. QSIP Message Type Registry

A new registry for QSIP Message Types is established. Each entry must include the Type Name (string) and a reference to its defining specification. Initial values are:

| Type Name | Reference |
|-----------|-----------|
| REGISTER | This document |
| PRESENCE | This document |
| INVITE | This document |
| ACCEPT | This document |
| REJECT | This document |
| END | This document |
| SESSION | This document |
| CHANNEL_UPDATE | This document |
| MESSAGE | This document |
| REACTION | This document |
| RECEIPT | This document |
| MESSAGE_ACTION | This document |
| INFO | This document |
| GROUP | This document |
| POLL | This document |
| QUEUE | This document |
| CALENDAR | This document |
| SCHEDULE | This document |
| ANALYTICS | This document |
| MODERATION | This document |
| AUDIT_EVENT | This document |
| ERROR | This document |
| KEY_EXCHANGE | This document |
| MEDIA_UPDATE | This document |
| USER_PROFILE | This document |
| SYNC | This document |
| PUSH | This document |
| SEARCH | This document |
| ROSTER | This document |
| CHANNEL | This document |
| BOT | This document |
| VOICEMAIL | This document |
| FEDERATION | This document |
| WORKSPACE | This document |
| STORY | This document |
| FORM | This document |
| PAYMENT | This document |
| TRANSCRIPT | This document |
| BACKUP | This document |
| MFA | This document |
| SUBSCRIPTION | This document |
| INTERACTIVE | This document |
| COMMAND | This document |
| WEBHOOK | This document |
| WORKFLOW | This document |
| MODAL | This document |
| CLIP | This document |

### 14.2. QSIP Error Code Registry

A new registry for QSIP Error Codes is established. Each entry must include the Code (number), Reason Phrase (string), and a reference. Initial values are defined in Section 9.2 of this document.

### 14.3. QSIP Info Type Registry

A new registry for Info Types used within the INFO message's info_type payload field.

| Info Type Name | Reference |
|----------------|-----------|
| typing_indicator | This document |

### 14.4. QSIP Action Type Registries

Separate registries for action fields within specific message types (e.g., SESSION, MESSAGE_ACTION, GROUP, POLL, QUEUE, CALENDAR, SCHEDULE, MODERATION, USER_PROFILE) are established. Initial values are those defined in the payload descriptions in Section 5.

## 15. Backwards-Compatibility and Interoperability

While QSIP is a new protocol, considerations for interworking with existing RTC ecosystems are important.

### 15.1. QSIP-to-SIP Gateway

A gateway can be implemented to translate between QSIP and SIP [RFC3261] domains.
Signaling: INVITE/ACCEPT/REJECT/END can be mapped to SIP equivalents. session_id maps to Call-ID. from.user_id maps to SIP From URI.
Media: Media would need to be transcoded or relayed if codecs/transports differ (e.g., QUIC Datagrams to SRTP over UDP). SDP from QSIP messages would be converted to standard SDP for SIP.
Presence: QSIP PRESENCE could be mapped to SIP PUBLISH/SUBSCRIBE/NOTIFY for presence (SIMPLE).
Challenges: Feature parity, especially for rich messaging and advanced QSIP types, will be difficult.

### 15.2. QSIP-to-XMPP Gateway

A gateway can map QSIP messaging and presence to XMPP [RFC6120], [RFC6121].
Messaging: QSIP MESSAGE can be mapped to XMPP <message> stanzas. Rich content and file URLs would need careful translation.
Presence: QSIP PRESENCE maps to XMPP <presence> stanzas.
Groups: QSIP GROUP actions map to XMPP Multi-User Chat (MUC) [XEP-0045] operations.
Calls: QSIP call signaling would need to be mapped to Jingle [XEP-0166] if audio/video interop is desired.

### 15.3. RTP Fallback Considerations

If a QSIP endpoint needs to interoperate with a legacy system that only supports RTP over UDP (not QUIC Datagrams), a media gateway (co-located with the QSIP/SIP or QSIP/XMPP gateway) would be required to transcode the media transport. The QSIP client itself would communicate using QSIP/QUIC to the gateway.

## 16. Multi-Device Synchronization

QSIP supports multi-device usage where a user can be simultaneously connected from multiple devices with the same user_id but different device_ids. This section describes how state and messages are synchronized across devices.

### 16.1. Device Management

Each device MUST have a unique device_id that remains consistent across sessions. The device_id is included in the from field of all messages sent by that device.

When a device registers using the REGISTER message, the server MUST:
* Track the device as active for that user
* Optionally send a SYNC message to other active devices notifying them of the new device
* Begin including this device in message distribution for the user

Devices can be explicitly managed using the USER_PROFILE message with appropriate actions to list, name, or remove devices.

### 16.2. Message Synchronization

When a message is sent to a user_id, the server MUST deliver it to all active devices for that user, with the following considerations:

* **Outgoing Messages**: When a user sends a message from one device, the server SHOULD echo it back to all other devices of the same user with a special flag or receipt indicating it was sent by another device.
* **Incoming Messages**: All devices receive incoming messages addressed to the user.
* **Read Receipts**: When a message is read on one device, a RECEIPT with status "read" SHOULD be sent to all other devices of the same user.
* **Message History**: New devices can request message history using the SYNC message type with appropriate parameters.

### 16.3. State Synchronization

Various types of state need to be synchronized across devices:

* **Presence State**: When presence is updated on one device, it SHOULD be reflected on all devices.
* **Call State**: Active calls can be transferred between devices using appropriate SESSION messages.
* **Group Membership**: Changes to group membership are automatically reflected on all devices.
* **User Profile**: Profile changes are synchronized to all devices via USER_PROFILE messages.

The SYNC message type facilitates explicit synchronization requests and responses between devices and the server.

## 17. Federation

QSIP supports federation between different QSIP servers, allowing users on different servers to communicate seamlessly. Federation uses the FEDERATION message type and follows specific procedures for cross-server communication.

### 17.1. Server Discovery

When a user attempts to communicate with a user on a different server (identified by a user_id containing a domain component, e.g., "alice@server1.example.com"), the originating server MUST:

* Extract the domain from the user_id
* Perform DNS lookups for QSIP service records (SRV records for _qsip._udp)
* Establish a server-to-server QUIC connection if not already established
* Authenticate the remote server using TLS certificates

### 17.2. Cross-Server Authentication

Server-to-server connections MUST use mutual TLS authentication with the following requirements:

* Each server MUST present a valid TLS certificate for its domain
* Certificates MUST be validated against trusted certificate authorities
* Servers MAY implement additional authentication mechanisms using the FEDERATION message type
* Once authenticated, servers exchange capability information including supported QSIP versions and features

### 17.3. Message Routing

When routing messages between federated servers:

* The originating server MUST validate that the sender is authorized to send from the claimed user_id
* Messages are forwarded with the original envelope intact, but MAY include additional routing headers
* The receiving server is responsible for local delivery to the target user
* Error messages MUST be routed back to the originating server
* Presence information MAY be shared between servers based on privacy settings and server policies

Federation enables building decentralized QSIP networks while maintaining the protocol's security and feature guarantees.

## 18. App Development

QSIP supports a rich ecosystem of third-party applications and bots that can extend the platform's functionality. This section describes how developers can create, distribute, and manage QSIP applications.

### 18.1. App Registration

Applications MUST be registered with a QSIP server or app directory before they can be used. Registration involves:

* **App Manifest**: A JSON document describing the app's capabilities, required permissions, commands, and interactive components.
* **OAuth Configuration**: Apps use OAuth 2.0 for installation and authorization.
* **Webhook URLs**: Endpoints for receiving events and interactions.
* **Bot User**: Most apps include a bot user that acts on behalf of the app.

Apps are identified by a unique `app_id` and authenticate using app-specific tokens separate from user tokens.

### 18.2. Permissions and Scopes

QSIP uses a granular permission system for apps:

**Common Scopes**:
* `messages:read` - Read messages in channels where installed
* `messages:write` - Send messages as the app
* `commands:write` - Register and handle slash commands
* `users:read` - Access basic user information
* `files:read` - Access files shared in workspace
* `webhooks:write` - Create and manage webhooks
* `channels:manage` - Create and modify channels
* `calls:write` - Initiate and manage calls

Apps MUST request only the minimum permissions needed and users MUST explicitly grant permissions during installation.

### 18.3. App Distribution

Apps can be distributed through:

* **Private Installation**: Direct installation via OAuth flow
* **Workspace App Directory**: Apps approved by workspace admins
* **Public App Directory**: Vetted apps available to all QSIP users
* **Enterprise Distribution**: Internal apps for specific organizations

The distribution method determines the review process and available features. Public apps undergo security review and MUST follow additional guidelines for data handling and user privacy.

## 19. JSON Schemas

Formal JSON Schema definitions for the QSIP message envelope and the payload of each message type defined in this document SHOULD be provided in a companion document or an appendix to ensure clarity for implementers and to enable automated validation. (Placeholder: Full JSON Schemas will be detailed in Appendix C or a separate linked document.)

## 20. References

### 20.1. Normative References

* **[RFC2119]** Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, DOI 10.17487/RFC2119, March 1997.
* **[RFC3550]** Schulzrinne, H., Casner, S., Frederick, R., and V. Jacobson, "RTP: A Transport Protocol for Real-Time Applications", STD 64, RFC 3550, DOI 10.17487/RFC3550, July 2003.
* **[RFC8126]** Cotton, M., Leiba, B., and T. Narten, "Guidelines for Writing an IANA Considerations Section in RFCs", BCP 26, RFC 8126, DOI 10.17487/RFC8126, June 2017.
* **[RFC8174]** Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words", BCP 14, RFC 8174, DOI 10.17487/RFC8174, May 2017.
* **[RFC8445]** Keranen, A., Holmberg, C., and J. Rosenberg, "Interactive Connectivity Establishment (ICE): A Protocol for Network Address Translator (NAT) Traversal", RFC 8445, DOI 10.17487/RFC8445, July 2018.
* **[RFC8446]** Rescorla, E., "The Transport Layer Security (TLS) Protocol Version 1.3", RFC 8446, DOI 10.17487/RFC8446, August 2018.
* **[RFC8489]** Petit-Huguenin, M., Salgueiro, G., Rosenberg, J., Wing, D., Mahy, R., and P. Matthews, "Session Traversal Utilities for NAT (STUN)", RFC 8489, DOI 10.17487/RFC8489, February 2020.
* **[RFC8656]** Reddy, T., Johnston, A., Matthews, P., and J. Rosenberg, "Traversal Using Relays around NAT (TURN): Relay Extensions to Session Traversal Utilities for NAT (STUN)", RFC 8656, DOI 10.17487/RFC8656, February 2020.
* **[RFC9000]** Iyengar, J., Ed. and M. Thomson, Ed., "QUIC: A UDP-Based Multiplexed and Secure Transport", RFC 9000, DOI 10.17487/RFC9000, May 2021.
* **[RFC9221]** Pauly, T., Kinnear, E., and D. Schinazi, "An Unreliable Datagram Extension to QUIC", RFC 9221, DOI 10.17487/RFC9221, March 2022.

### 20.2. Informative References

* **[RFC3261]** Rosenberg, J., Schulzrinne, H., Camarillo, G., Johnston, A., Peterson, J., Sparks, R., Handley, M., and E. Schooler, "SIP: Session Initiation Protocol", RFC 3261, DOI 10.17487/RFC3261, June 2002.
* **[RFC6120]** Saint-Andre, P., "Extensible Messaging and Presence Protocol (XMPP): Core", RFC 6120, DOI 10.17487/RFC6120, March 2011.
* **[RFC6121]** Saint-Andre, P., "Extensible Messaging and Presence Protocol (XMPP): Instant Messaging and Presence", RFC 6121, DOI 10.17487/RFC6121, March 2011.
* **[RFC7519]** Jones, M., Bradley, J., and N. Sakimura, "JSON Web Token (JWT)", RFC 7519, DOI 10.17487/RFC7519, May 2015.
* **[draft-ietf-avtcore-rtp-over-quic]** Engelbart, M., et al., "RTP over QUIC", Work in Progress, Internet-Draft.
* **[draft-ietf-sframe]** époque, E., et al., "Secure Frame (SFrame)", Work in Progress, Internet-Draft. (Note: Exact draft name may vary, this is illustrative)
* **[XEP-0045]** Saint-Andre, P., "Multi-User Chat", XMPP Standards Foundation.
* **[XEP-0166]** Ludwig, S., et al., "Jingle", XMPP Standards Foundation.

## Appendix A: Example Flows

(Textual descriptions of common flows. PlantUML or similar diagrams would be ideal in a full draft.)

### A.1. Peer-to-Peer Audio/Video Call Setup

1. Client A (Alice) -> Server: REGISTER
2. Server -> Client A: RECEIPT (status: "registered")
3. Client B (Bob) -> Server: REGISTER
4. Server -> Client B: RECEIPT (status: "registered")
5. Alice -> Server: INVITE (to: Bob, session_id: S1, media: audio/video, ICE candidates, SFrame offer)
6. Server -> Bob: INVITE (from: Alice, session_id: S1, media, ICE, SFrame offer - forwarded)
7. Bob -> Server: ACCEPT (to: Alice, session_id: S1, selected media, ICE candidates, SFrame answer)
8. Server -> Alice: ACCEPT (from: Bob, session_id: S1, media, ICE, SFrame answer - forwarded)
9. Alice <-> Bob: (ICE connectivity checks complete - P2P or via TURN)
10. Alice -> Server: KEY_EXCHANGE (to: Bob, session_id: S1, SFrame key parameters for sending to Bob)
11. Server -> Bob: KEY_EXCHANGE (from: Alice, to: Bob, session_id: S1, SFrame key parameters)
12. Bob -> Server: KEY_EXCHANGE (to: Alice, session_id: S1, SFrame key parameters for sending to Alice)
13. Server -> Alice: KEY_EXCHANGE (from: Bob, to: Alice, session_id: S1, SFrame key parameters)
14. Alice <-> Bob: Encrypted media (SFrame over QUIC Datagrams or SRTP) flows.
15. Alice -> Server: END (to: Bob, session_id: S1, reason: "hangup")
16. Server -> Bob: END (from: Alice, session_id: S1, reason: "hangup")

### A.2. Group Text Chat

1. Alice -> Server: GROUP (action: "create", name: "Team Chat", members: [Bob, Charlie]) -> group_id: G1 returned
2. Server -> Bob: GROUP (action: "invited_to_group", group_id: G1, inviter: Alice) (or similar notification)
3. Server -> Charlie: GROUP (action: "invited_to_group", group_id: G1, inviter: Alice)
4. Alice -> Server: MESSAGE (to: group_id: G1, content: "Hello team!")
5. Server -> Alice: RECEIPT (target_message_id: M_Alice1, status: "delivered_to_server")
6. Server -> Bob: MESSAGE (from: Alice, to: group_id: G1, content: "Hello team!")
7. Server -> Charlie: MESSAGE (from: Alice, to: group_id: G1, content: "Hello team!")
8. Bob -> Server: INFO (to: group_id: G1, info_type: "typing_indicator", is_typing: true)
9. Server -> Alice: INFO (from: Bob, to: group_id: G1, ...)
10. Server -> Charlie: INFO (from: Bob, to: group_id: G1, ...)
11. Bob -> Server: MESSAGE (to: group_id: G1, content: "Hi Alice!")
12. Server -> (Distributes Bob's message to Alice, Charlie; sends receipts)

### A.3. Call Center Queue

1. Customer -> Server: QUEUE (action: "enqueue", queue_id: "support", details: {...})
2. Server -> Customer: RECEIPT (target_message_id: M_Cust1, status: "enqueued", payload: { item_id: I1, position: 5, est_wait: 300s })
3. (Time passes, agents become available)
4. Server -> AgentA: QUEUE (action: "offer_item", item_id: I1, customer_id: Customer, details: {...})
5. AgentA -> Server: QUEUE (action: "agent_accept", item_id: I1, agent_id: AgentA)
6. Server -> Customer: SESSION (action: "agent_connecting", agent_display_name: "Agent A") (or an INVITE for a call)
7. Server -> AgentA: INVITE (to: Customer, session_id: S2, ...)
8. (Call proceeds like P2P call)

## Appendix B: Detailed State Machines

(Placeholder: This section would contain formal state machine diagrams (e.g., UML or PlantUML source) for key QSIP entities like Sessions, Presence, Group Membership, and Queue Items. For brevity in this combined document, detailed diagrams are omitted but would be essential for a full IETF draft.)

Key stateful entities include:

* **Client Registration State**: Unregistered, Registering, Registered, RegistrationFailed.
* **Session State (Call/Conference)**: IDLE, INVITING, RINGING (INCOMING/OUTGOING), CONNECTING, ACTIVE, ON_HOLD, RECONNECTING, TERMINATING, TERMINATED.
* **Message State (for reliable messaging with receipts)**: UNSENT, SENT_TO_SERVER, DELIVERED_TO_RECIPIENT_SERVER, DELIVERED_TO_RECIPIENT_CLIENT, READ, FAILED.
* **Queue Item State**: PENDING, OFFERED_TO_AGENT, ASSIGNED_TO_AGENT, ACTIVE_WITH_AGENT, RESOLVED, ABANDONED.

## Appendix C: Comparative Analysis

This table summarizes how the finalized QSIP specification addresses common RTC needs compared to existing WebRTC setups and commercial telecom APIs like Twilio, assuming QSIP is fully implemented as specified.

| Capability | WebRTC (Baseline) | Twilio (Typical Commercial API) | QSIP (as specified) |
|------------|-------------------|----------------------------------|---------------------|
| Signaling Protocol | Undefined (App-specific: SIP, XMPP, custom WebSocket) | Proprietary REST/WebSocket APIs, SIP | Unified JSON over QUIC |
| Transport | UDP for media (RTP), TCP/WebSocket for signaling | UDP/TCP (various internal) | Single QUIC connection (UDP) for signaling & media datagrams |
| Connection Setup Latency | Higher (separate handshakes) | Optimized, but often multi-protocol | Very Low (QUIC 0-1 RTT) |
| NAT Traversal (Media) | ICE/STUN/TURN (Complex) | Managed (Global Relay Network) | ICE/STUN/TURN (Simplified by single QUIC port) |
| Security (Transport) | DTLS-SRTP (media), TLS (signaling) | TLS (API), SRTP (media) | QUIC (TLS 1.3) for all traffic by default |
| End-to-End Encryption (Media) | Optional (SFrame emerging) | Optional/Varies | SFrame explicitly supported via KEY_EXCHANGE |
| Rich Messaging Features | Not inherent (App-specific) | Yes (SMS, Chat APIs) | Native: Text, files, reactions, threads, receipts, edit/delete, polls, etc. |
| Presence | Not inherent (App-specific) | Yes (e.g., TaskRouter agent status) | Native PRESENCE and USER_PROFILE messages |
| Group Management | Not inherent (App-specific) | Yes (Video Rooms, Chat Channels) | Native GROUP messages |
| Call Center / Queuing | Not inherent | Yes (TaskRouter, Flex) | Native QUEUE messages |
| Codec Flexibility | Good (Opus, VP8/9, H264, AV1 emerging) | Good (Opus, VP8, H264) | Excellent (Opus, AV1 recommended), flexible negotiation |
| Simulcast/SVC | Supported in implementations | Supported | Supported explicitly (signaled via capabilities/media negotiation) |
| Extensibility | High (via application logic) | Via APIs and webhooks | High (Custom message types, payload extensions, IANA registries) |
| Browser Native Support | Yes (for WebRTC media/data channels) | Yes (SDKs abstract underlying tech) | Requires QUIC-enabled environment (native apps, servers, modern browsers with QUIC API, or WebAssembly polyfill for some aspects) |
| PSTN Integration | Via external gateways | Core feature | Via Gateways (QSIP-to-SIP, as specified in Sec 15) |
| Global Media Relay | App/Service dependent | Core feature | Infrastructure dependent; protocol supports TURN for relaying. |
| Developer Simplicity | Moderate-High (signaling complexity) | Good (SDKs, but API surface can be large) | Aimed for High (Unified protocol, JSON, clear message types) |
| Standardization | Core media is IETF/W3C | Proprietary API | Proposed Open Standard (IETF-style draft) |

**QSIP Advantages Highlighted by the Specification:**

* **Unified & Simplified Stack**: Single protocol over a single port reduces complexity significantly.
* **Performance**: Lower latency and potentially better resource utilization due to QUIC.
* **Security by Default**: TLS 1.3 for all communications is a strong baseline.
* **Comprehensive Feature Set**: Natively supports a broad range of RTC and collaboration features often requiring multiple protocols or services.
* **Open & Extensible**: Designed as an open standard with clear extensibility points.

## Authors' Addresses

QSIP Working Group (Conceptual)
Email: contact@example-qsip-wg.org (Illustrative)

---

End of QSIP Specification Document
