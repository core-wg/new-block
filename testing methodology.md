# Overview

This draft handles the situation where there is a <strong>need to send data using CoAP that does not fit into a single packet, but does not need/expect a confirming response</strong>. As a reminder, RFC7959 extends RFC7252 for handling data that spans multiple packets, but is only suitable for Confirmable interchanges.

It was mandatory for this draft that, using UDP with the possibility of no feedback that can be used to determine network or peer overload, <strong>the necessary guards (i.e., congestion control) were in place to prevent aggressive sending rates along with support for recovery that did not cause any instability</strong>. These considerations are detailed in the applicability scope and further in the congestion control section.  

# Requirement Background

The DOTS signal protocol ([RFC8782](https://datatracker.ietf.org/doc/rfc8782/)) allows DOTS agents in the Internet to request mitigation support against DDoS attacks.  This is achieved using CBOR for the data encoding, carried using CoAP-over-DTLS, and the data sizes reduced as much as possible by having the CBOR variables as indices into a variable name array, etc.

When no attack is ongoing, the initial DOTS signal channel configuration negotiation is done using Confirmable messages, the subsequent signal information is sent using Non-Confirmable as there are likely to be packet losses due to DDoS attacks. The DOTS application layer has the necessary built-in recovery for any lossy traffic.

Infrequent bulk data transfers are done over the DOTS data channel ([RFC8783](https://datatracker.ietf.org/doc/rfc8783/)) using HTTPS and RESTCONF.  It is recognized that this will likely fail when network connectivity is affected by DDoS attacks. <em>Interop testing revealed that the DOTS data channel fails to be established during attack time, while the signal channel continues to work as expected (see this [report](https://datatracker.ietf.org/meeting/103/materials/slides-103-dots-interop-report-from-ietf-103-hackathon-00)).</em>

Following interoperability tests and general DOTS community feedback, it became apparent that it would be very useful to have additional information exchanged when DDoS attacks were taking place – to provide real-time feedback to the entities doing the DDoS mitigation so that methods/algorithms, etc. could be tuned to maximize the DDoS target’s availability to the Internet to non-DDoS users. [draft-ietf-dots-telemetry](https://datatracker.ietf.org/doc/html/draft-ietf-dots-telemetry) was created to define the telemetry requirements to handle this real-time feedback information.

DOTS interoperability tests highlighted that this telemetry information must use Non-confirmable, and that it was not possible to fit sufficient telemetry information into a single packet.  Hence the creation of draft-ietf-core-quick-block to be able to handle the missing CoAP piece, namely support for multiple packets for a body using Non-confirmable.

# Core Meetings Handling draft-ietf-core-quick-block

A design goal was to use as much of the existing CoAP RFCs functionality as possible, and any work done would not require the need to update any of the existing CoAP RFCs. <strong>To date there has been no need for any updating of existing CoAP RFCs.</strong>

During these core WG meetings, potential design challenges ware raised and mutually agreed solutions were suggested. Some of these solutions turned out to be unworkable, but triggered other solutions which worked when handling Congestion Control, Recovery, etc. This is captured in the WG presentations and minutes (see next).

## Core Meetings Presentations and Minutes

* 2021-03-10 IETF 110 [Presentation](https://datatracker.ietf.org/meeting/110/materials/slides-110-core-sessa-draft-ietf-core-quick-block-01.pdf) [Minutes]( https://datatracker.ietf.org/meeting/110/materials/minutes-110-core-202103121300-01)
* 2020-11-17 IETF 109 [Presentation](https://datatracker.ietf.org/meeting/109/materials/slides-109-core-sessa-draft-ietf-core-new-block-01.pdf) [Minutes](https://datatracker.ietf.org/doc/minutes-109-core-202011201600/)
* 2020-10-22 Interim Core [Presentation](https://datatracker.ietf.org/meeting/interim-2020-core-11/materials/slides-interim-2020-core-11-sessa-coap-block-wise-transfer-options-for-faster-transmission-draft-ietf-core-new-block-01-00) [Minutes](https://datatracker.ietf.org/meeting/interim-2020-core-11/materials/minutes-interim-2020-core-11-202010221600-00)
* 2020-07-28 IETF 108 [Presentation](https://datatracker.ietf.org/meeting/108/materials/slides-108-core-sessb-coap-block-wise-transfer-options-for-faster-transmission-00.pdf) [Minutes](https://datatracker.ietf.org/doc/minutes-108-core-202007281410/)
* 2020-06-10 Interim Core [Presentation](https://datatracker.ietf.org/meeting/interim-2020-core-06/materials/slides-interim-2020-core-06-sessa-coap-block-wise-transfer-options-for-faster-transmission-01.pdf) [Minutes](https://www.ietf.org/proceedings/interim-2020-core-06/minutes/minutes-interim-2020-core-06-202006101600-00)
* 2020-05-13 Interim Core [Presentation](https://datatracker.ietf.org/meeting/interim-2020-core-04/materials/slides-interim-2020-core-04-sessa-new-coap-block-wise-transfer-options-draft-bosh-core-new-block-01.pdf) [Minutes](https://datatracker.ietf.org/doc/minutes-interim-2020-core-04-202005131600/)

# DOTS Interoperability Testing

Interoperability testing was key for checking out DOTS implementations and throwing up any potential misunderstanding of the drafts issues. The primary interoperability testing was done between a proprietary DOTS solution (NCC) and an open source implementation [go-dots](https://github.com/nttdots/go-dots).

The later interoperability tests covered DOTS telemetry (draft-ietf-dots-telemetry). Initially this was done with both ends using RFC7959 (requests were Non-Confirmable which RFC7959 cautions about using) and highlighted some implementation issues of draft-ietf-dots-telemetry as well as the RFC7959 implementation, all of which were fixed and draft-ietf-dots-telemetry text tightened.  Use of RFC7959 was stable apart from handling packet loss.

draft-ietf-core-quick-block was added to the proprietary DOTS solution, but has not yet been added to go-dots. Interoperability tests continued, requests for Q-Block support were rebuffed and both sides continued using RFC7959 (with Non-confirmable). As there were no packet losses, all behaved as expected.

The latest Interop was carried out in Feb 2021 with the Q-Block enabled at the proprietary implementation.
Separately, interoperability tests were completed in an environment where there was packet loss (restricted pipe, higher DDoS attack data rates), but this was done before draft-ietf-dots-telemetry. <strong>It validated the need for real-time signal channel information to be sent using Non-Confirmable as well as the DOTS applications were able to handle the situation with the attacks being successfully mitigated.</strong>

# draft-ietf-core-quick-block Testing
## DOTS

The propriety DOTS implementation has a lot of regression tests which include testing out the functionality of DOTS client’s communication with DOTS servers (both signal and data) and includes handling the DOTS telemetry information.  They do include handling uni-directional packet loss and the restarting of the DOTS agents to verify the expected recoveries, etc. do occur.  They do not test out draft-ietf-core-quick-block per se, but do handle some (minor amount) traffic that is split out over multiple packets.  The introduction of draft-ietf-core-quick-block code did not affect any of the regression testing results.

## Implementation Testing using Libcoap

Libcoap is used for the CoAP implementation for the proprietary DOTS implementation.  RFC7959 is handled within the libcoap layer and the body of the data is presented to the application layer.
draft-ietf-core-quick-block is also implemented within the libcoap layer (see https://github.com/obgm/libcoap/pull/611) 

There are a lot of libcoap regression test suites for checking out libcoap.
These compare output from a known good run with the current run to highlight
changes/issues. These regression tests can take hours - diagnostic tools such
as valgrind are being used to check for memory leaks, memory access errors etc.

These are not included in the libcoap distributions as a good run output varies
between build environments (e.g., different autotool messages, required
security by later TLS libraries (e.g., warnings about SHA1), etc.).  As
the known good run outputs are different, a lot of potential false positives are
created in different build environments.

These regression test do not modify any of the base CoAP parameters (e.g.,
ACK_TIMEOUT).

There is a specific regression test suite where traffic uses (Q-)Block1 for large requests and (Q-)Block2 for large responses along with Observe large responses.  The block SZX was also made smaller in some cases to stimulate many packets being required for a single body. The tests cover Confirmable, Non-Confirmable, and TCP traffic but do not include any packet loss. The Clients cycle through using BlockX and Q-BlockX (which may fail, reverting back to BlockX) and the servers cycling through only supporting BlockX or supporting both BlockX and Q-BlockX to test out all 4 combinations.

Running this regression test suite with a libcoap that does not support Q-BlockX has exactly the same results when run against a libcoap that does have Q-BlockX support.  The libcoap that does not support Q-BlockX just ignores the request to test for Q-BlockX support and carries on with BlockX.  This shows that functionality (without any recovery needed) is the same for the two option types.  <strong>This demonstrates stability in the implementation and the draft</strong>.

It should be noted however that this regression test suite initially highlighted that some sort of ‘Continue’ was needed as the NON_TIMEOUT delay after MAX_PAYLOADS packets was causing some of the tests to timeout without all of the data being received.  Fixing ‘Continue’ in the draft and implementing fixed these failing regression tests. <strong>‘Continue’ only happens when the peer has received the appropriate data and so the network and peer are not overloaded.</strong>

Libcoap provides the testing capability of one or more specific packets dropped or a packet randomly dropped. Testing was done with one or more packets missing from the start, around the MAX_PAYLOADS pause point, at the end of the body as well as in the middle of a MAX_PAYLOADS sequence for both CON and NON.  Lessons learnt about, e.g., the recoveries triggered updates to the draft. <strong>Recovery for any packet loss combination was fully tested out with no instability detected.</strong>

Libcoap was tested with complete packet loss in one direction (for NON) confirming data still gets through, albeit more slowly with the NON_TIMEOUT timeout for every MAX_PAYLOADS packets adding to the overall transmission time. <strong>Note that CON fails under these conditions, and as such RFC7959 fails to retrieve the body.</strong>

Multiple concurrent clients were run in parallel - including with enforced packet loss to verify correct ‘session’ separation in the server.

Client and Servers were stopped mid traffic flow (peer remaining up) and restarted to test out that things recovered as expected.

All these tests were repeated many times. 

It is intended to make the above additional tests that are not currently in a regression test suite a part of a regression test suite for ongoing implementation/stability tests.

# Misc
## DOTS Interoperability Tests

* https://datatracker.ietf.org/meeting/106/materials/slides-106-dots-dots-telemetry-related-hackathon-activity-report-00
* https://datatracker.ietf.org/meeting/105/materials/minutes-105-dots-01
* https://datatracker.ietf.org/meeting/104/materials/slides-104-dots-interoperability-and-hackathon-report-00
* https://datatracker.ietf.org/meeting/103/materials/slides-103-dots-interop-report-from-ietf-103-hackathon-00
* https://datatracker.ietf.org/meeting/102/materials/slides-102-dots-ietf-102-hackathon-interop-report-00
* https://datatracker.ietf.org/meeting/101/materials/slides-101-dots-ietf-101-hackathon-dots-interop-01
* https://datatracker.ietf.org/meeting/100/materials/slides-100-dots-hackathon-and-interoperability-test-report-00


