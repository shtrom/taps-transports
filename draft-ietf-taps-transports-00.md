---
title: "Services provided by IETF transport protocols and congestion control mechanisms"
abbrev: TAPS Transports
docname: draft-ietf-taps-transports-00
date: 2014-11-14
category: info
ipr: trust200902

author:
 -
    ins: G. Fairhurst
    name: Godred Fairhurst
    role: editor
    org: University of Aberdeen
    street: School of Engineering, Fraser Noble Building
    city: Aberdeen AB24 3UE
    email: gorry@erg.abdn.ac.uk
 - 
    ins: B. Trammell
    name: Brian Trammell
    role: editor
    org: ETH Zurich
    email: ietf@trammell.ch
    street: Gloriastrasse 35
    city: 8092 Zurich
    country: Switzerland
 - 
    ins: M. Kuehlewind
    name: Mirja Kuehlewind
    role: editor
    org: ETH Zurich
    email: mirja.kuehlewind@tik.ee.ethz.ch
    street: Gloriastrasse 35
    city: 8092 Zurich
    country: Switzerland

normative:
  RFC0791:
  RFC2460:
informative:
  RFC0768:
  RFC0793:
  RFC0896:
  RFC4340:
  RFC4960:
  RFC5348:
  RFC5405:

--- abstract

This document describes services provided by existing IETF protocols and congestion control mechanisms.  It is designed to help application and network stack programmers and to inform the work of the IETF TAPS Working Group.

--- middle

# Introduction

Most Internet applications make use of the Transport Services provided by TCP (a reliable, in-order stream protocol) or UDP (an unreliable datagram protocol). We use the term "Transport Service" to mean the end-to-end service provided to an application by the transport layer. That service can only be provided correctly if information is supplied from the application. The application may determine the information to be supplied at design time, compile time, or run time, and may include guidance on whether an aspect of the service is required, a preference by the application, or something in between. Examples of (aspects/components) of Transport Services are reliable delivery, ordered delivery, content privacy to in-path devices, integrity protection, and minimal latency.

The IETF has defined a wide variety of transport protocols beyond TCP and UDP, such as TCP, SCTP, DCCP, MP-TCP, and UDP-Lite. Transport services may be provided directly by these transport protocols, or layered on top of them using protocols such as WebSockets (which runs over TCP) or RTP (over TCP or UDP). Services built on top of UDP or UDP-Lite typically also need to specify a congestion control mechanism, such as TFRC or the LEDBAT congestion control mechanism.  This extends the set of available Transport Services beyond those provided to applications by TCP and UDP.

Transport protocols can aslo be differentiated by the aspects of the services they provide: for instance, SCTP offers a message-based service that does not suffer head-of-line blocking when used with multiple stream, because it can accept blocks of data out of order, UDP-Lite provides partial integrity protection when used over link-layer services that can support this, and LEDBAT can provide low-priority "scavenger" communication.

# Terminology

The following terms are defined throughout this document, and in subsequent documents produced by TAPS describing the composition and decomposition of transport services.

The terminology below is that as was presented at the TAPS WG meeting in Honolulu. While the factoring of the terminology seems uncontroversial, thre may be some entities which still require names (e.g. information about the interface between the transport and lower layers which could lead to the availablity or unavailibility of certain transport protocol features)

Transport Service (Component/Aspect): 
:a specific feature a transport service provides to its clients end-to-end. Examples include confidentiality, reliable delivery, ordered delivery, message-versus-stream orientation, etc.

Transport Service: 
:a set of transport service (components/aspect), without an association to any given framing protocol, which provides a complete service to an application.

Transport Protocol: 
:an implementation that provides one or more different transport services using a specific framing and header format on the wire.

Transport Protocol Feature: 
:an implementation of a transport service (component/aspect) within a protocol

Transport Service Instance: 
:an arrangement of transport protocols with a selected set of features and configuration parameters that implements a single transport service, 
e.g. a protocol stack (RTP over UDP)

Application: 
:an entity that uses the transport layer for end-to-end delivery data across the network.

# Existing Transport Protocols

(An introductory matrix table goes here)

## Template: Imaginary Transport Protocol (ITP)

(This subsection is a rough template provide initial guidance to subsection authors on how a subsection should be structured and what information should be provided. It will be removed once a real transport protocol has been fully described in the draft; that protocol's description can then be used as template for the others.)

### Introduction and Applicability

(This subsection should be at most three paragraphs: what's the protocol called, where can one read about it, what does it do, where is it deployed, and what applications is it used for.)

The Imaginary Transport Protocol (ITP) [reference] provides a unidirectional, connectionless, message-oriented, partially-reliable transport for fixed-sized messages over IP, using protocol number 257. It is intended primarily for use in bidirectional constant-bit-rate (CBR) unicast media transmission. It is not widely implemented within the Internet, as I just made it up, and it has several fatal design flaws, including requiring an illegal protocol number for its operation.

### Base Features

(This subsection should describe each base feature of the base protocol which may be interesting for decomposing the service(s) it provides. Base features are those which require no (1) optional API interactions in normal usage or (2) additional headers on the wire. Dividing descriptions of features into subsections is desirable but optional, snce these features may depend on each other. Each feature should be covered by about two paragraphs.)

ITP's features include time- and retransmission-limited partial reliability and constant-sized message-oriented framing. All ITP messages are 392 octets long, with 8 bytes of header and 384 bytes of payload. Each message has a message sequence number; ITP acknowledgment messages are 16 octets long. Each message will be retransmitted up to two times (for three total transmissions)if not acknowledged, or if not acknowledged within an application-defined time. Further details of ITP's acknowledgment and message sequence number management scheme are found in [reference].

ITP provides no additional congestion control features, since the limited retransmission facility was deemed incapable of contributing to congestion collapse. 

### Optional Features

(This subsection should describe each optional feature of the protocol which may be interesting for decomposing the service(s) it provides, as above.)

ITP's retransmission can be optionally completely disabled at the sender-side, though the receiver will continue to send acknowledgments. This optional operation mode has no other effect

### Application Interface Considerations

(This subsection should describe features the protocol requires of the API. How is the protocol, along with its optional features, normally used by the application? Does it use the sockets API; if so, are there socket options?)

ITP can be used via the sockets interface on systems which support it, by using the SOCK_IMAGINARY socket type. Regardless of the size of the buffer passed to sendmsg(2), the message will be truncated or zero-padded to 384 octets in the ITP message, and recvmsg(2) always returns 384 bytes.

No information about the ack stream is available; retransmission is disabled via setsockopt(2).

### Interactions with other protocols

(If the protocol uses other protocols to provide additional service components -- commonly, this would include (D)TLS encapsulation -- that should be noted here)

ITP can be encapsulated within DTLS to provide confidentiality, integrity, and authentication, as described in [reference].

### Candidate Components

(This subsection should list candidate components derived from the features above.)

- Time- and retransmission-limited reliability
- Unidirectional transmission
- Fixed-length message-oriented transmission

## Transmission Control Protocol (TCP)

(Volunteer: Mirja Kuehlewind)

TCP is a reliable transport protocol that is widely used in the Internet.
Reliability is implemented by sending an acknowledgment (ACK) on the reception of a data segment and retransmitting (potentially) lost data when no new acknowledgment is received. 
For this purpose a data segment can be identified by a sequence number (SEQ). 
Each new payload byte increases the SEQ by one. 
The ACK contains, morerover, an acknowledgment number that announces the next in-order SEQ expected by the receiver. 
To reduce signaling overhead, a TCP receiver might not acknowledge each received segment separately but multiple at once. 
A TCP receiver should at least acknowledge every second packet and delay an acknowledgement not more that 500ms [rfc5681, rfc1122]. 
<!-- Most operating systems implement a maximum delay of 100ms today. There are hardware implementation for high speed networks that accumulate even more. 
Loss is assumed by the sender if three duplicated ACKs are received or no ACK is received for a certain time.--> 
Duplicated ACKs are triggered at the receiver by the arrival of out-of-order data segments and thereby do not acknowledge new data but repeat the previous acknowledgment number.  
<!-- If this data does not carry the next expected SEQ, the receiver will send out an ACK with the same acknowledgment number again. 
This is called a duplicated ACK. --> 
When the missing data is received, a cumulative acknowledgment is sent that acknowledges all (now) in-order segments received so far.
Additionally, TCP often implements Selective Acknowledgment (SACK) [rfc2018], where, in case of duplicated ACKs, the received sequence number ranges are announced to the sender.
Therefore when more than one packets was lost, the sender does not have to wait until an accumulated ACK announces the next whole in the sequence number space, but can retransmit lost packets immediately.


When a loss is detected,  TCP congestion control [rfc5681] becomes active and usually reduces the sending rate.
The sending rate is most often determined by a (sliding-) window. 
Based on the packet conservation principle [Jacobson1988], new packets are only sent into the network when an old packets has exited. 
This leads to TCP implementations that are mostly self-clocked and take actions only when an ACK is received.
When no ACKs are received at all for a certain time larger than at least one  RTT, the RTO is triggered and the sending rate is reduced to a minimum value.
The current sending window is the minimum of the receive window and the congestion window where % ($cwnd$) which maintains the number of packets which are currently allowed to be in flight.
the receive window is announced by the receiver to not overload the receiver buffer.
As long as flow control does not become active and not signal a smaller receive window to not overload the receiver, the sending window equals the congestion window.
The congestion window is estimated by the congestion control algorithm based on implicit or explicit network feedback.
<!--In general, congestion control is based on some kind of network feedback.-->
Inherently, the congestion control mechanism of a TPC sender assumes that loss occurs to due network congestion und therefore reacts each time congestion is notified.
Note, congestion control usually at most reduces  once per  RTT as the congestion feedback has a signaling delay of one  RTT. 
Unfortunately, congestion is not the only reason for the occurrence of loss. 
Packets can be dropped for various reasons by different nodes on the network path, e.g due to bit errors.
<!--In the Internet the non-congestion based loss rate is usually very low, thus it is common sense to always handle loss as congestion signal.-->

<!-- delay, explicit signal -> thus connection-oriented like TCP. -->
<!--Other mechanisms exist that try to avoid congestion losses and thus reduce the sending before a queue overflows by sensing an increase in end-to-end delay.
Another opportunity can be realized based on network-support where network node explicitly provide a congestion notification, e.g. based in ECN.
In any case, these congestion signal will be seen by the receiver and the transport protocol needs to provide a way to feed back this information to the sender that actually can control the sending rate.
-> probing for newly available capacity induces (periodically) congestion... -->

To determine e.g. a large enough value for the RTO, the  RTT needs to be measured by a  TCP sender.
The RTTM mechanism in TCP either needs to store the sent-out time stamp and SEQ  of all or a sample of packets or can use the TSOpt [rfc1323]. 
With TSOpt the sender adds the current time stamp to each packet and the receiver reflects this time stamp in the respective ACK. 
By subtracting the reflected time stamp from the current system time the  RTT can be measured for each received ACK holding a valid TSOpt.
Note that in case of delayed or duplicated ACKs, the time stamp of the first unacknowledged packet is reflected which might inflate the  RTT measurement artificially but guarantees a large enough RTO.
<!--"Then a single subtract gives the sender an accurate RTT measurement for every ACK segment (which will correspond to every other data segment, with a sensible receiver)." [rfc1323]-->


## User Datagram Protocol

(Volunteer: Kevin Fall)

## Realtime Transport Protocol

(Volunteer: Varun Singh)

## Stream Control Transmission Protocol

(Volunteers: Michael Tuexen, Karen Nielsen)

# Transport Service Components

(drawn from the candidate components in the previous section)

# Matrix of Protocols and Components

(a comprehensive matrix table goes here; Volunteer: Dave Thaler)

# IANA Considerations

This document has no considerations for IANA.

# Security Considerations

This document surveys existing transport protocols and protocols providing transport-like services; the (components/aspects) of those services include confidentiality, integrity, and authenticity. 

# Acknowledgments



