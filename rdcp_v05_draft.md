# RDCP v0.5 specification draft

- RDCP is the ROLORAN Disaster Communication Protocol.
- This document covers v0.5 of RDCP as an early draft in late 2025.
- RDCP is currently in development and experimentation as part of the EU-funded (#NextGenerationEU) [dtec.bw research project ROLORAN](https://dtecbw.de/home/forschung/unibw-m/projekt-roloran) in cooperation with [Gemeinde Neuhaus, Austria](https://neuhaus.gv.at/).
- This document shortly summarizes the most important aspects of RDCP for implementers and interested third parties.
- v0.5 will be the third generation of RDCP, improving scalability and robustness over v0.4 based on practical experiences, lab setups and simulations made throughout 2025.

## RDCP Fundamentals

- `RDCP Messages` are transmitted as the payload of LoRa packets.
- Each `RDCP Message` consists of an `RDCP Header` followed by an `RDCP Payload`.
- The `RDCP Header` has a fixed size. It includes a checksum to verify correct reception independent of underlying (LoRa) mechanisms.
- The size of the `RDCP Payload` depends on the `RDCP Message Type` used and can vary between 0 bytes (i.e., no payload) and (200 bytes - `RDCP Header`size), i.e., the LoRa packet payload must not exceed 200 bytes. The 200 bytes limit is chosen arbitrarily based on observations of how robust LoRa transmissions are with different LoRa parameters and hardware implementations in real-world scenarios. In general, shorter messages should be preferred over longer ones.
- Sensitive `RDCP Message Types` use cryptographic signatures or authenticated encryption to enable the verification of the `Origin` of the `RDCP Message` and keep the payload content confidential. As signatures based on asymmetric cryptography have a significant payload size impact, they are deliberately omitted from selected `RDCP Message Types`.

## RDCP Infrastructures and Basic RDCP Terms

- RDCP applies orchestrated flooding of messages by clocked relaying in a two-tier mesh/star network topology.
- `RDCP Messages` are flooded in a mesh of `RDCP Infrastructure` backbone nodes referred to as "Digitale Anschlagtafeln" (`DAs`, digital bulletin boards, also known as `MERLIN Bases`), typically using one common 433 MHz LoRa channel.
- In the vicinity of each `DA` there may be mobile devices ("Mobile Geräte"/`MGs`, also known as `MERLIN Messengers`). `DAs` use an 868 MHz LoRa channel to communicate with `MGs`.
- Each `RDCP Infrastructure` has headquarters (`HQ`, "Krisenstab"). `HQ` devices are operated by public authorities and emergency services and have additional, privileged functionality at their disposal, such as sending `Official Announcements` (`OAs`) to facilitate authorities' communication with citizens. Typically only one `HQ` device is active at any given time, others may be running in a passive/stand-by mode in which they only receive, but do not actively send `RDCP Messages`.
- An `RDCP Infrastructure` as a whole is the set of all RDCP-capable devices in one `Scenario`. `HQ` devices and `DAs` are managed by an `RDCP Infrastructure Operator`, which also prepares/configures and hands out `MGs` to `end-users` (citizens) in a provisioning/roll-out/deployment process. A `scenario` is the deployment of RDCP devices in a real-world environment, test environment, or simulation.
- At any point in time, an `RDCP Infrastructure` operates in either `crisis mode` or `non-crisis mode`. In both modes, the `HQ` devices can send new `OAs` to `DAs` and `MGs`, but only in `crisis mode` the `DAs` and `MGs` can also be used to send `Citizen Reports` back to `HQ`. Information about which mode the `RDCP Infrastructure` is currently in is distributed with selected `RDCP Message Types`; devices lacking this information, e.g., after power-on without any received `RDCP Messages` yet, assume that the `RDCP Infrastructure` is in `crisis mode` as a fail-safe choice.

## Neuhaus Scenario

- The first larger-scale real-world deployment of an `RDCP Infrastructure` is the municipality of Neuhaus, Austria. It is called `MERLIN` (Messaging with regional LoRa infrastructure).
- In the `Neuhaus Scenario` for 2025, there are 10 `DAs` and around 100 `MGs` as constrained by project resources.
- `DAs` work as both mesh `Relays` and devices that can be used interactively (the latter is similar to the `MGs`).
- `MGs` are implemented using LilyGo T-Deck hardware with a custom-made firmware prototyped in the ROLORAN research project.
- `DAs` use custom-made hardware prototyped in the ROLORAN project with a focus on energy efficiency and autarchy for longer-term total black-out crisis scenarios.
- The `HQ` devices use custom-made software prototyped in the ROLORAN project running on COTS notebooks, using T-Deck `MG` hardware via USB as "LoRa modem" to connect to nearby `DAs` on the 868 MHz channel.
- The limited maturity of any ROLORAN artifacts including specifications, software implementations and hardware prototypes reflects the status of a research project and its available resources. Contributions by volunteers are welcome, as is cooperation and joint development interest by third parties with or without commercial interest.
- The Neuhaus Scenario currently uses RDCP v0.4 as a longer-term field deployment; RDCP v0.5 leverages the experiences made in this challenging real-world scenario to improve scalability and robustness in order to enhance the applicability to other scenarios that may be larger or more complex given their topology.

## RDCP Addresses and DA Identifiers

- RDCP uses 16-bit addresses as a trade-off between compact headers and support for a sufficiently large number of devices in the overall `RDCP Infrastructure`. We currently consider it unlikely that RDCP will be used in scenarios with more than 64k devices within the next few years; thus, scaling to a larger address space, e.g., 32-bit and beyond, may be considered later, as are other ideas, such as interconnecting regional `RDCP Infrastructures` by other means.
- An individual `RDCP address` is assigned to each device.
- Certain `RDCP Messages`, such as `OAs`, can be sent to the `RDCP broadcast address`, i.e., everyone.
- `RDCP multicast addresses` are used for groups of devices.
- Each RDCP device 'listens' to its individual `RDCP address`, the `RDCP broadcast address`, and the `RDCP multicast addresses` of the device groups it is assigned to. `DAs` can serve as `Relays` for any other `RDCP Messages` as well.
- RDCP's 16-bit address space is currently used as follows:

| RDCP address/range | Usage                          |
| -----------------: | :----------------------------- |
| 0x0000             | *reserved* for device-internal use |
| 0x0001 - 0x00FE    | `HQ` devices                   |
| 0x00FF             | `HQ` multicast address         |
| 0x0100 - 0x01FF    | *reserved* for backward compatibility |
| 0x0200 - 0x02FF    | `DAs`                          |
| 0x0300 - 0xAEFF    | `MGs` for citizens             |
| 0xAF00 - 0xAFFF    | `MGs` for tests and exercises  |
| 0xB000 - 0xBFFF    | Group/multicast addresses      |
| 0xC000 - 0xFEFF    | `MG` s for special purposes    |
| 0xFF00 - 0xFFFE    | *reserved*                     |
| 0xFFFF             | Broadcast address              |

- Based on the namespace, there could be a maximum of 256 `DAs`. We limit current scenarios to 250 `DAs` and use the lower byte of their `RDCP Address` as their 8-bit `Identifier`. For example, the `Identifier` of 0x0215 is 0x15. The term `Identifier` is used in the context of `Relay Identifiers` and `Entry Point Identifiers` below. `Identifier` values in the range 251-255 (0xFB - 0xFF) can serve as magic values and do not persistently identify one specific `DA`.

## RDCP Header

- The `RDCP Header` has a fixed size of 15 bytes.
- An `RDCP Message` consists of an `RDCP Header` followed by an `RDCP Payload` (if there is any payload for the used `RDCP Message Type`; otherwise, the whole `RDCP Message` consists of only the header). The maximum `RDCP Payload` size ("*max*") is 200 bytes - the `RDCP Header` size.
- Important nomenclature:
  - The device (currently) transmitting an `RDCP Message` is referred to as `Sender`.
  - The device that created an `RDCP Message` and initially transmitted it is referred to as `Origin`.
  - Thus, an `RDCP Message` only has one `Origin` but there can be many `Senders` when the message is spread throughout the topology.
  - The first `DA` to send a new `RDCP Message` for distribution in the mesh network on the 433 MHz LoRa channel is referred to as `Entry Point`. `MGs` typically choose their nearest `DA` as `Entry Point` but getting the message flooded in the mesh may fall back to alternatives if a specific `DA` (the `MG's` primary `Entry Point`) is currently unavailable. `DAs` serve as their own `Entry Points` for locally created messages (e.g., through interactive `DA` use or when automatically responding to incoming messages, such as telemetry requests).
  - Once the `Entry Point` has transmitted the `RDCP Message`, specific other `DAs` serve as `Relays` to propagate the `RDCP Message` in the mesh network. The goal is that each `RDCP Message` can be received by each `DA` at least once, even if there may be `DAs` and `MGs` in their vicinity for which the particular `RDCP Message` is not relevant. More selective routing that does not flood the whole topology may optionally be implemented.
  - To avoid packet collisions, each `DA` has a `routing table` carrying information about when to send which `RDCP Messages`. These `routing tables` can be updated remotely by `HQ` devices to reflect changes in the overall topology or routing strategy, e.g., if new `DAs` are set up or a `DA` runs out of power and is unavailable for an unknown or longer time.
  - Additionally, `RDCP Messages` can be transmitted multiple times by the same `Sender` in a row to compensate for lower LoRa packet delivery rates. The `Retransmission Counter` in the `RDCP Header` indicates how often an `RDCP Message` will be sent again by the same `Sender` in the same `Timeslot` (see below). The number of retransmissions (i.e., the initial value of the `Retransmission Counter`) is specified for each `RDCP Message Type` and must be used by RDCP-compatible devices if the message is to be relayed or forwarded. `RDCP Messages` not intended for relaying may use different (usually lower) initial values for their `Retransmission Counter`.
  - The actual duration (in wallclock time) of one `Timeslot` is calculated as *nrt* * (*airtime* + *1000 ms*), where *nrt* is the (initial) number of retransmissions for the `RDCP Message Type`, *airtime* is the LoRa packet time-on-air (which depends on the overall LoRa packet size and the LoRa parameters chosen in the `Scenario`, such as coding rate, spreading factor, preamble length, and bandwidth), and *1.000 ms* are currently used as a buffer time between retransmissions and subsequent `timeslots` to cover processing times in the used low-energy embedded systems.
  - The overall duration it takes for an `RDCP Message` to be transmitted by its `Entry Point` and `Relays` on the 433 MHz channel is referred to as `propagation cycle duration`. RDCP devices, especially `Entry Points`, are supposed to delay sending a new message at least until the `propagation cycle` of the previous message has finished to avoid error situations such as packet collisions and message reordering; see the section about RDCP Relaying and Flooding for details.
  - The actual length (in number of `timeslots`) and therefore duration (in wallclock time) of a `propagation cycle` depends on the number of `Relays` used for the specific `RDCP Message`. In turn, this depends on the `Entry Point` of the `RDCP Message` and typically its position in the scenario's topology. Each device can derive the actual length from the information in its `routing table` (see `RDCP Message Type` `Routing Information`).
- The `RDCP Header` consists of:

| Header field name        | Field size in bits | Description                                                                           |
| ------------------------ | :----------------: | ------------------------------------------------------------------------------------- |
| `Checksum`               | 16                 | CRC-16 (CCITT) checksum of all subsequent `RDCP Header` fields and the `RDCP Payload` |
| `Sender`                 | 16                 | `RDCP address` of `Sender`                                                            |
| `Origin`                 | 16                 | `RDCP address` of `Origin`                                                            |
| `Sequence Number`        | 24                 | Message `Sequence Number` (specific for `Origin`)                                     |
| `Destination`            | 16                 | `RDCP address` of message recipient                                                   |
| `Entry Point`            |  8                 | `Entry Point Identifier`, i.e. lower byte of the `Entry Point's` `RDCP Address`       |
| `Message Type`           |  8                 | `RDCP Message Type`                                                                   |
| `Payload Length`         |  8                 | Size of `RDCP Payload` in bytes                                                       |
| `Timeslot`               |  4                 | `Timeslot` number this message is currently sent in                                   |
| `Retransmission Counter` |  4                 | `Retransmission Counter` for currently transmitted message                            |

- The `Checksum` is calculated for the whole `RDCP Message` (i.e., `RDCP Header` and `RDCP Payload`) except the 16-bit `Checksum` field itself.
- The `Sequence Number` is specific for the `Origin` of the `RDCP Message` and must be incremented (usually by 1) for each new message, starting at 1, e.g., when the device is initially provisioned.
- The `Message Type` values, valid ranges for the `Payload Length`, and initial `Retransmission Counter` values for all `RDCP Message Types` are specified below.
- `Timeslot` indicates the timeslot in the `propagation cycle` that the message is currently sent in. An `Entry Point` sends in `Timeslot` 0.
- Note that at most 15 `Relays` and retransmissions can be used based on the 4-bit value range for both fields.
- Fields larger than 8 bits are sent in little-endian order, i.e., the lowest byte comes first and the highest byte is last.
- `MGs` use the `Entry Point` field to indicate which `DA` is expected to serve as `Entry Point` on the 868 MHz channel. The `Entry Point` field is preserved when propagating the message over the 433 MHz channel and plays a key role in routing. The magic value of 0xFF can be used to indicate that the message must not be relayed. For messages created at or by a `DA` itself, it serves as its own `Entry Point` (unless 0xFF is used).

## RDCP Message Types

This section specifies the `RDCP Message Types` currently in use.

### RDCP Message Types Overview

In the following table,

- *sig* represents the size of a cryptographic signature (depending on implementation, see section RDCP Message Authentication and Encryption).
- *tag* represents the size of an AES-256-GCM Authentication Tag (16 bytes).
- the number of retransmissions within each `timeslot` is specified as *none* (i.e., 0), *low*, *medium*, or *high*. The actual values should be chosen based on packet delivery rates (PDR) in the scenario, e.g., *low* = 0, *medium* = 2, and *high* = 4. Retransmissions increase the likelihood that at least one instance of the repeated message is properly received. If a message with a *low* number of retransmissions is lost, nothing really bad happens to the overall `RDCP Infrastructure`. Missing a message with a *medium* number of retransmissions typically causes increased network load because messages will be sent again or the lost information may cause minor inconsistencies in the overall `RDCP Infrastructure`. If a message with a *high* number of retransmissions is not successfully propagated throughout the `RDCP Infrastructure`, important information such as an `Official Announcement` may not reach its destination and this information loss might not be detected for messages that do not require explicit acknowledgments or responses. On the other hand, the number of retransmissions is directly proportional to the duration of the `propagation cycle`, so unnecessary high values should be avoided in the interest of channel capacity.
- *max* refers to the maximum `RDCP Payload` size (200 bytes - `RDCP Header` size).

| Value | Name                      | Retransmissions | Payload size       | Payload content        | Short description                                 |
| ----: | :---                      | :-------------: | -----------------: | :--------------        | :----------                                       |
| 0x00  | `TEST`                    | *low*           | 0 - *max* bytes    | plain text             | Reserved for test messages                        |
| 0x01  | `ECHO REQUEST`            | *none*          | 0 bytes            | none                   | Simple unicast connectivity test ("ping")         |
| 0x02  | `ECHO RESPONSE`           | *none*          | 0 bytes            | none                   | Response to echo request                          |
| 0x05  | `DA STATUS REQUEST`       | *low*           | 4 bytes + *tag*    | binary data            | Telemetry data request for a `DA`                 |
| 0x06  | `DA STATUS RESPONSE`      | *low*           | >= 8 bytes + *tag* | binary data            | Telemetry data response from a `DA`               |
| 0x09  | `BLOCK DEVICE ALERT`      | *low*           | 4 bytes + *sig*    | binary data, signature | Exclude a device from relaying                    |
| 0x0A  | `TIMESTAMP`               | *low*           | 6 bytes + *sig*    | binary data, signature | Date/time/mode information from an `HQ`           |
| 0x0B  | `DEVICE RESET`            | *low*           | 2 bytes + *sig*    | `Nonce`, signature     | Reset instruction for a single device             |
| 0x0C  | `DEVICE REBOOT`           | *low*           | 2 bytes + *sig*    | `Nonce`, signature     | Reboot instruction for a single device            |
| 0x0D  | `MAINTENANCE PREP`        | *none*          | 2 bytes + *sig*    | `Nonce`, signature     | Switch single device to maintenance mode          |
| 0x0E  | `RESET OF INFRASTRUCTURE` | *medium*        | 2 bytes + *sig*    | `Nonce`, signature     | Reset of all participating devices                |
| 0x0F  | `ACKNOWLEDGMENT`          | *medium*        | >= 3 bytes + *sig* | binary data, signature  | Acknowledgment that message was received          |
| 0x10  | `OFFICIAL ANNOUNCEMENT`   | *high*          | 6 - *max* bytes    | binary/Unishox2 data   | Official announcement by an `HQ`                  |
| 0x11  | `RESET OF OFF. ANN.`      | *medium*        | *sig*              | signature              | Revoke all previous `OFFICIAL ANNOUNCEMENTs`      |
| 0x1A  | `CITIZEN REPORT`          | *high*          | >= 3 bytes + *tag* | binary/Unishox2 data   | Message from `MG` to `HQ`                         |
| 0x20  | `FETCH ALL NEW MESSAGES`  | *none*          | 2 bytes            | `Sequence Number`      | `DA` fetches all new messages from other `DA`     |
| 0x21  | `FETCH MESSAGE`           | *none*          | 2 bytes            | `Reference Number`     | `DA` fetches specific message from other `DA`     |
| 0x2A  | `DELIVERY RECEIPT`        | *none*          | 0 bytes            | none                   | Signal that response to 0x20/0x21 is completed    |
| 0x30  | `CRYPTOGRAPHIC SIGNATURE` | *high*          | 2 bytes + *sig*    | binary data, signature | Separate signature for `OFFICIAL ANNOUNCEMENT`    |
| 0x31  | `HEARTBEAT`               | *none*          | >= 4 bytes (+ *tag*) | binary data            | Heartbeat / device registration                   |
| 0x32  | `RTC`                     | *low*           | >= 4 bytes + *sig* | binary data, signature | `DA`-specific `RTC`                               |
| 0x35  | `ROUTING INFORMATION`     | *medium*        | >= 3 bytes + *tag* | binary data            | Routing table update information by `HQ` to `DA`  |
| 0x36  | `ROUTING CONFIRMATION`    | *medium*        | 5 bytes + *tag*    | binary data            | Routing table confirmation by `DA` to `HQ`        |
| 0x37  | `SEQUENCE NUMBER OVERRIDE`| *medium*        | *n* * 5 bytes + *sig* | binary data         | `HQ` sets `Sequence Numbers` to use               |
| 0x38  | `SEQUENCE NUMBER ALARM`   | *medium*        | 2 bytes + *tag*    | binary data            | Device reports wrong `Sequence Number` to `HQ`    |
| 0x40  | `TUNNELED MESSAGE`        | given by payload| 2 - *max* bytes    | binary data            | Arbitrary data tunneled over RDCP                 |

### RDCP Test messages

RDCP `TEST` messages are *reserved* for testing and debugging purposes and not used during regular operation of `RDCP Infrastructures`. They are forwarded/distributed by `Relays` but not automatically interpreted by any device. The `RDCP Payload` consists of 0-*max* bytes of plain text, i.e., no encoding, compression, or encryption is used. `Entry Points` may apply rate limiting and drop `TEST` messages without further notice. Support for `TEST` messages may be disabled completely in production environments, i.e., they may be ignored by recipients and not relayed by `DAs`.

### RDCP Echo Request and Echo Response messages

RDCP `ECHO REQUEST` and `ECHO RESPONSE` messages are similar to their ICMP equivalents in the Internet Protocol world. An RDCP-capable device is expected to respond with an `ECHO RESPONSE` message when it receives an `ECHO REQUEST` message unless it is configured to ignore them. In combination, both `RDCP message types` allow for a simple check of the connectivity/reachability of the target device, i.e., the `Destination` of the `ECHO REQUEST` and the `Origin` of the `ECHO RESPONSE`.

Both `RDCP message types` have an empty `RDCP Payload`. An `ECHO REQUEST` message may only be sent to a single target device (unicast). `ECHO REQUEST` messages sent to multicast/group addresses or the `RDCP broadcast address` are always dropped (not forwarded) by `Relays` and should not be answered by receiving devices.

Sending an `ECHO REQUEST` is usually triggered manually using an `HQ` device or a dedicated management `MG` by the `RDCP Infrastructure Operator` to verify the target device's basic connectivity, e.g., when a `DA` appears to not be working properly. Software implementations for `DA` and `MG` devices should not support sending `ECHO REQUESTs` through their primary end-user interfaces. `Entry Points` may refuse to propagate both types of messages, e.g., based on rate limiting. Support for both `RDCP message types` may be disabled completely in production environments.

### RDCP DA Status Request and DA Status Response messages

RDCP `DA STATUS REQUEST` and `DA STATUS RESPONSE` messages handle `DA` telemetry data.

A `DA STATUS REQUEST` message is usually sent once by the `HQ` via unicast after the `DA` has been powered on. `DAs` then send `DA STATUS RESPONSE` messages and optionally `HEARTBEAT` messages at regular intervals. If a `DA` is restarted, e.g., after maintenance or power loss, a new `DA STATUS REQUEST` is sent by the `HQ`. The `HQ` should time and configure its `DA STATUS REQUEST` messages in a way that `DA STATUS RESPONSE` and `HEARTBEAT` messages of all `DAs` in a scenario are spread evenly across time intervals, e.g., to have half of the total number of `DAs` send their `DA STATUS RESPONSE` messages within one hour. Suitable timing values should be chosen based on LoRa parameters and resulting channel capacity as well as `HQs'` demand for up-to-date information in a specific scenario.

Both `RDCP message types` use AES-256-GCM encryption and authentication. The described `RDCP Payloads` are replaced with the AES ciphertext and the GCM AuthTag is appended.

Payload of `DA STATUS REQUEST`:

- (8 bit) Delay in minutes before first `DA STATUS RESPONSE` is sent
- (8 bit) Delay in minutes between `DA STATUS RESPONSE` messages, e.g., 120 (2 hours); magic value 0 disables sending `DA STATUS RESPONSEs`.
- (8 bit) Delay in minutes before first `HEARTBEAT` is sent
- (8 bit) Delay in minutes between `HEARTBEAT` messages, e.g., 30 (half an hour); magic value 0 disables sending `HEARTBEATs`.
- (*tag*) AES-256-GCM AuthTag

Payload of `DA STATUS RESPONSE`:

- (16 bit) Battery status (8 bit per battery)
- (16 bit) Number of unique received `RDCP Messages` since the last `DA STATUS RESPONSE`
- (16 bit) Number of unique relayed/forwarded `RDCP Messages` since the last `DA STATUS RESPONSE`
- (16 bit) Number of different `MG` devices from which messages were received since last `DA STATUS RESPONSE`
- n * (32 bit) List of other `DA` devices that were received by this `DA` when they were the `Sender` of an `RDCP Message` since the last `DA STATUS RESPONSE`: (16 bit) Other `DA`'s `RDCP Address`, (8 bit) RSSI value, (8 bit) SNR value. The RSSI and SNR values are implementation-specific and can be taken from the most recently received message by the other `DA` or be an average value.
- (*tag*) AES-256-GCM-AuthTag

`DA STATUS RESPONSE` messages are always sent to the `HQ multicast address` as `Destination`, not the `Origin` of the original request. In a `DA STATUS RESPONSE`, other/neighboring `DAs` may be assigned virtual `RDCP addresses` to distinguish between 433 MHz and 868 MHz channels. For example, 0x0200 may be used to indicate a 433 MHz link to the `DA` with `RDCP address` 0x0200, while 0x0100 may be used to indicate an 868 MHz link to the same `DA`.

### RDCP Device blocking message

 `HQ` devices can use `BLOCK DEVICE ALERT` messages to instruct `Relays` to drop (not forward) `RDCP Messages` originating from a specific device. It is intended for situations in which a `DA` or `MG` is clearly misused, e.g., to submit a larger amount of fake `CITIZEN REPORTs`.

The payload consists of:

- (16 bit) `RDCP Address` of the blocked device
- (16 bit) Blocking duration in minutes. The magic value of 0x0000 indicates that the device shall not longer be blocked.
- (*sig*) Cryptographic signature (message must be sent by an `HQ` device)

Obviously, `DAs` should only be blocked in extreme situations as they might be used by different end-users. `Entry Points` may apply additional measures to prohibit the use of unused/spoofed `Origin` `RDCP Addresses` independent of specific device blocking.

`BLOCK DEVICE ALERT` messages can be sent to individual devices (usually `DAs`), multicast groups, or the broadcast address. Untampered `MGs` can honor `BLOCK DEVICE ALERTs` that affect them by temporarily disabling their sending capability and primary user interfaces related to sending messages.

### RDCP messages related to Device and Infrastructure Maintenance

`DA` devices are operated 24/7/365 and eventually require maintenance operations due to software updates, insufficient solar energy and power failures, or software-side deadlocks.

The `RDCP Messages` in this category prepare a single device, typically a `DA`, or the `RDCP Infrastructure` part consisting of all `DAs`, for these maintenance operations. These messages may only be sent by an `HQ` or dedicated maintenance devices, as verified via cryptographic signatures.

An RDCP `DEVICE RESET` message instructs a single device, usually a `DA`, to perform a software-side reset of itself. This typically includes, among other actions, the deletion of volatile and selected persisted data, such as displayed `Official Announcements`, `RDCP Message` duplicate tables, and the queue of scheduled `RDCP Messages` to be sent. However, other persisted data, such as the last used own `Sequence Number` and own `RDCP Address`, must not be wiped. It is left to device implementation how this operation is carried out in detail.

The `RDCP Payload` of a `DEVICE RESET` message consists of:

- (16 bit) `Nonce`
- (*sig*) Cryptographic signature

The `Nonce` (a number used only once) must be increased (usually by 1) for each message of this type and avoids replay attacks because messages of this type are not checked for duplicates based on the `Sequence Number` (wrong `Sequence Numbers` are one of the historic problems to be solved with device resets). The target device therefore must also persist the most recently used value of this `Nonce` (in combination with the `Origin` unless global nonce values are used in the scenario).

An RDCP `DEVICE REBOOT` message is similar to a `DEVICE RESET`, but expects the target device to completely restart, including an operating system reboot (if applicable), and re-initialize all hardware components. The `RDCP Payload` consists of a `Nonce` and a cryptographic signature as above.

An RDCP `RESET OF INFRASTRUCTURE` message is also similar to a `DEVICE RESET`, but affects all devices that receive it, not only a single one, i.e., the `Destination` usually is the broadcast address. In addition, receiving devices reset their own used `Sequence Number`. However, `Relays` still have to forward the message before resetting themselves if they are able to do so. The `RDCP Payload` consists of a `Nonce` and a cryptographic signature as above. Messages of this type are expected to reach all `DAs`, but the majority of `MG` devices should be considered to be offline. It is sort of an operation "of last resort" if there was a major incident with the technical infrastructure that cannot be recovered from on a device-by-device basis. Note that a "real" full reset of the `RDCP infrastructure` in a scenario includes a rollover of cryptographic key material to prevent replay attacks; this currently involves manual configuration/provisioning of the involved RDCP devices.

Finally, an RDCP `MAINTENANCE PREPARATION` message turns on the maintenance mode for a single target device. In maintenance mode, the device can enable additional interfaces, e.g., for USB, Ethernet, WiFi, or Bluetooth access, or over-the-air-updates. For `DAs`, this mode is usually activated in combination with physical access to the device. Such `RDCP Messages` may be sent by an `HQ` or dedicated maintenance `MG` devices and their `RDCP Payload` consists of a `Nonce` and cryptographic signature, similar to `DEVICE RESET`, `DEVICE REBOOT`, and `RESET INFRASTRUCTURE`. Maintenance always ends with a reboot of the target device to manually verify that it properly starts up also in the case of a temporary power failure. If maintenance mode has been activated accidentally, it can remotely be ended with a `DEVICE REBOOT` message. Support for `MAINTENANCE PREPARATION` messages is optional; devices may alternatively provide, for example, a physical switch for this purpose or temporarily enable maintenance access interfaces after a reboot.

Implementations of how messages in this maintenance category are handled by the target device(s) are expected to treat them with priority, in the sense that they still should work even if "nothing else works anymore". If they fail, physical access to the devices is required to, for example, power-cycle them.

### RDCP Timestamp and Mode Information messages

Currently, `DAs` and regular `MGs` do not have battery-buffered real-time clocks. They can be powered off for an arbitrary amount of time (e.g., during the night if a `DA`'s battery is empty, until there is enough solar energy during the day again) and therefore have no persistant internal knowledge about the real-world wallclock time. RDCP primarily addresses this issue by using durations, e.g., the `Message Lifetime` of an `Official Announcement`, instead of absolute start and end dates and times.

RDCP `TIMESTAMP` messages can be sent, likely periodically, by an `HQ` device to distribute the current date and time in the `RDCP Infrastructure`. It is not intended as a time synchronization protocol and given the time it takes to propagate `RDCP Messages` via the `Relays`, no noteworthy accuracy is achieved. However, sometimes it can be useful to have absolute timestamps in internal logfile or display the current time to `end-users` of a device, even if it is off by a minute.

The `RDCP Payload` of `TIMESTAMP` messages is deliberately kept primitive to avoid that device implementations need to handle details such as leap years and can parse it trivially:

- (8 bit) number of years since 2025 (0 = 2025, 1 = 2026, ...)
- (8 bit) month (1 = January, ..., 12 = December)
- (8 bit) day of month (0, ..., 31)
- (8 bit) hour (0, ..., 23)
- (8 bit) minute (0, ..., 59)
- (8 bit) `RDCP Infrastructure Mode` information:
  - 0x00: `RDCP Infrastructure` is currently in `non-crisis mode`
  - 0x01: `RDCP Infrastructure` is currently in `crisis mode` and `HQ` is staffed
  - 0x02: `RDCP Infrastructure` is currently in `crisis mode` but `HQ` is unmanned
- (*sig*) Cryptographic signature

### RDCP Acknowledgment messages

RDCP `ACKNOWLEDGMENT` messages confirm that another (the confirmed) `RDCP Message` has been successfully received by the `Origin` of the `ACKNOWLEDGMENT`. Acknowledgments are used in the context of messages created by `end-users` (`RDCP Message Type` `CITIZEN REPORT`) and only acknowledge that the confirmed message was received on the technical communication level, not that it has been read and processed by a human (yet).

In the most simple case, `ACKNOWLEDGMENT` messages have the `Origin` of the confirmed message as `Destination` and the following `RDCP Payload`:

- (16 bit) `Sequence Number` of the acknowledged `RDCP message`
- (8 bit) `Type of acknowledgment`:
  - Value 0x00: `Positive acknowledgment` - the acknowledged message was technically received and will be processed further.
  - Value 0x01: `Negative acknowledgment` - the acknowledged message was technically received, but cannot be processed further. This is intended for situations in which `end-users` send a `CITIZEN REPORT` message, but the overall `RDCP Infrastructure` is not in `crisis mode` and therefore no-one at an `HQ` device will be able to quickly react manually. Official `DA` and `MG` devices disable the sending of such messages when they are certain to be outside of a crisis situation, but depending on which messages those devices do and do not receive or when they are powered on, they may consider themselves in `crisis mode` although the rest of the `RDCP Infrastructure` is not.
  - Value 0x02: `Positive negative acknowledgment` - the acknowledged message was technically received and will be processed further, but the `HQ` is currently unmanned.
- (*sig*) Cryptographic signature

The active `HQ` device and the `Entry Point` **must** respond to incoming `CITIZEN REPORT` messages with an `ACKNOWLEDGMENT`. The cryptographic signature is only mandatory for `HQ` devices currently, an `Entry Point` may omit it from the acknowledgment message. Messages to be acknowledged must have a unicast address as their `Origin`. Messages that have other `Origin` addresses are not acknowledged.

In the case that an `ACKNOWLEDGMENT` is sent by an `HQ` device, multiple other messages may be acknowledged with a single message (`Multi-ACK`). A `Multi-ACK` message is sent to the `RDCP broadcast address` as `destination` and its payload consists of a list of

- (16 bit) `RDCP address` of the device for which another message is acknowledged
- (16 bit) `Sequence Number` of the device's acknowledged `RDCP message`
- (8 bit) `Type of acknowledgment` as above

along with a cryptographic signature under consideration of the maximum `RDCP payload size`. `Multi-ACKs` can be used by `HQs` if they receive multiple `CITIZEN REPORT` messages within a short timeframe so that multiple outgoing `ACKNOWLEDGMENTs` would be queued while waiting for a free channel; those acknowledgments can then be merged into a `Multi-ACK` in the interest of channel capacity and response latency.

### RDCP messages related to Official Announcements

In `RDCP Infrastructures`, `Official Announcements` are text-based information prepared by a public authority or emergency service to be displayed on `DA` and `MG` devices. An `OFFICIAL ANNOUNCEMENT` message must be sent from an `HQ` device and can be addressed to either of:

- Everyone: The `Destination` address is set to the `RDCP broadcast address`.
- A group of devices: The `Destination` address is set to the corresponding `RDCP multicast address`.
- A single device: The `Destination` address is set to the target device's `RDCP Address`.

The payload of an `OFFICIAL ANNOUNCEMENT` (`OA`) consists of:

- (8 bit) `Subtype` of the `OA` message:
  - 0x10: `Non-crisis mode` announcement, for example information about a public event outside of a crisis situation
  - 0x20: `Crisis mode` announcement, for example information on the current situation and ongoing aid efforts
  - 0x22: `Message lifetime update`, used to change the lifetime of a previously sent `OA`
  - 0x30: `Feedback` to a `CITIZEN REPORT`, e.g., the information that help for a life-threatening emergency is on the way
  - 0x31: Further `inquiry` about a `CITIZEN REPORT`: Same as 0x30, but the recipient is expected to answer with another `CITIZEN REPORT`
  - 0x32: `Unsolicited inquiry`: Same as 0x31 but without reference to a previous `CITIZEN REPORT`
- (16 bit) `Reference Number`:
  - For subtype 0x10, 0x20, and 0x32 `OAs`, the `Reference Number` is a fresh `Nonce` that can be used to refer back to this `OA` later
  - For subtype 0x22 `OAs`, the `Reference Number` identifies the previously sent `OA` whose lifetime should be changed
  - For subtype 0x30 and 0x31 `OAs`, the `Reference Number` is the `Reference Number` of the related `CITIZEN REPORT`
- (16 bit) `Message Lifetime`; the initial lifetime for a new `OA` or the new lifetime for a previously sent `OA`:
  - A lifetime of 0 indicates that the `OA` should be deleted/invalidated. Only used with subtype 0x22 `OAs`.
  - Values in the range between 1 and 60000 specify the lifetime in minutes.
  - Values in the range between 60001 and 65534 specify the lifetime in full days (24h). For example, 60002 means 2 days, 60010 means 10 days.
  - A lifetime of 65535 (largest possible value in the unsigned 16-bit range) indicates infinite lifetime. Such messages can be explicitly deleted with a subtype 0x22 `OA`.
- (8 bit) `More Fragments` (see below, only relevant for `OAs` sent to the `RDCP broadcast address`)
- (0 - (*max* - 6/22 bytes) `Content` of a new `OA` (see below) or (*sig*) Cryptographic signature for a `Subtype` 0x22 `OA`

`OAs` may be short, but typically contain longer text to keep the citizens well-informed with the necessary level of detail. For this reason, `OFFICIAL ANNOUNCEMENT` messages have the following specific characteristics:

- The textual `Content` of an `OA` is compressed/encoded using Unishox2, an algorithm dedicated to compressing and decompressing short texts. It should include a human-readable timestamp of when the `OA` was created/sent by the `HQ` device.
- If the Unishox2-compressed text does not fit into a single `OA` message, the original announcement text is split into the necessary number of parts. These parts are then referred to as "fragments" and each fragment is sent in an `OFFICIAL ANNOUNCEMENT` message of its own. All these fragments share the same `Reference Number`, and the recipient must use the `More Fragments` value to bring the fragments into the correct order. For example, if an `OA` is split into three fragments, the first fragment has a `More Fragments` value of 2, the second fragment has a `More Fragments` value of 1, and the final fragment has a `More Fragments` value of 0. Fragments must also be sent in this order, so the recipient knows from the beginning whether more fragments are to be expected and can also identify whether fragments are missing towards the end of the assembled `Content`. Multiple fragments can be used only for `OAs` sent to the `RDCP broadcast address`; `OAs` directed at single devices or multicast groups must always fit into a single `RDCP Message`. Note that each fragment must be properly Unishox2-decodable on its own; thus, when splitting a longer text into fragments, the original (uncompressed) text must be split so that its Unishox2-compressed representation does not exceed the size limit for each fragment.
- `OFFICIAL ANNOUNCEMENT` messages to the `RDCP broadcast address` are accompanied by separate `CRYPTOGRAPHIC SIGNATURE` messages to authenticate their `Origin`. For multi-fragment `OAs` only a single `CRYPTOGRAPHIC SIGNATURE` message is used (detailed below in the section about `CRYPTOGRAPHIC SIGNATURE` messages). The `CRYPTOGRAPHIC SIGNATURE` message may be sent before or after the `OA` (or all of its fragments). `DAs` and `MGs` should appropriately inform `end-users` about `OAs` if a cryptographic signature is still missing or its verification failed.
- If the `HQ` device has a shared secret (cryptographic key material) established with the `Destination` device (unicast) or group (multicast), the complete `RDCP Payload` is encrypted and authenticated after the `Content` is Unishox2-compressed, with the `static` `RDCP Header` fields as additional authenticated data (see below). In this case, there is no accompanying `CRYPTOGRAPHIC SIGNATURE` message, but the 16-byte AES GCM AuthTag must be considered regarding maximum message length.

Implementations must ensure by proper splitting into fragments that the maximum `Content` length is not exceeded.

An `RESET OF OFFICIAL ANNOUNCEMENT` message may be sent by an `HQ` device to trigger the deletion of all currently displayed/stored `OAs` on `DAs` and `MGs`. It is typically used at the end of crisis exercises or real crises to start fresh without removing all the other data that is deleted via the maintenance-category `RDCP Messages`. The `RDCP Payload` consists only of a cryptographic signature.

### RDCP messages for Citizen Reports

Only during a crisis, `DAs` and `MGs` as `end-user` devices can be used to send text messages to the `HQ multicast address`. Either the most recently received `OFFICIAL ANNOUNCEMENT` along with its `Subtype` or the most recently received `TIMESTAMP` is typically used to derive whether the `RDCP Infrastructure` is in `crisis mode` (`OA` `Subtype` 0x10) or `non-crisis mode` (`OA` `Subtype` 0x20). However, one practical consideration is to always start `end-user` devices in `crisis mode` so that urgent emergencies can be reported immediately after powering the devices on without having to wait for a new `OA`/`TIMESTAMP` or a periodic retransmission of recent `OAs`. It thus cannot be reliably avoided that `CITIZEN REPORTs` are also transmitted in `non-crisis mode`, at least on the 868 MHz channel.

The user interfaces for `end-user` devices currently distinguish between three types of `CITIZEN REPORTs`, and they typically use different forms that have to be filled in manually with different data.

`Subtype` `EMERGENCY` messages are for reporting urgent, severe events involving life-threatening casualties that require assistance of highest priority to save lives, such as medivac of a person in a hard-to-reach crisis area. `End-users` have to fill in the typical emergency report questions of "who reports?", "where is the emergency?", "what happened?" and "how many persons are affected?". The `Content` of the `RDCP Payload` of a `Subtype` `EMERGENCY` message concatenates the answers to these questions separated by the `#` symbol and is Unishox2-compressed. Additionally, default values can be encoded with short magic values, e.g., if the person who reports is the same whom an `MG` has been provisioned for.

`Subtype` `CITIZEN REQUEST` messages are used for still important, yet slightly less urgent reports by `end-users`. The spectrum of potential reports is broad, ranging from observations of damaged public infrastructure (such as destroyed roads) and private property (damaged houses and flooded cellars) to a request to have groceries delivered anyhow because households in a crisis area are running out of food and fresh water during a longer crisis. `End-users` typically have to answer "who reports?", select a type of request or observation from a list, and can enter a short text (about 100-150 characters) to describe their situation. Similar to above, the `Content` of the `RDCP Payload` of `Subtype` `CITIZEN REQUEST` messages concatenates this input and Unishox2-compresses it.

`Subtype` `RESPONSE TO INQUIRY` messages can only be sent if the `end-user` device previously received an `OFFICIAL ANNOUCEMENT` of `Subtype` 0x31 or 0x32. In such situations, the personnel at the active `HQ` device needs further information about a previous `CITIZEN REPORT` or have other questions about the recipient's current situation, and `end-users` can fill in a free-text field in response. This `Content` is also Unishox2-compressed.

More `Subtypes` may be added on demand for specific scenarios, but the user interfaces should be kept as simple as possible. As with `Official Announcements`, `Citizen Reports` should be kept short and to the point, and low-entropy chatter should be discouraged to keep the channels free for important messages; for this reason, RDCP does not provide an MG-to-MG chat capability and instead encourages the use of alternatives such as Meshtastic or citizen band radio.

`CITIZEN REPORT` `RDCP Messages` always have the `HQ multicast address` as their `Destination`.

The `RDCP Payload` of `CITIZEN REPORT` messages consists of:

- (8 bit) `Subtype`:
  - 0x00 for `EMERGENCY`
  - 0x01 for `CITIZEN REQUEST`
  - 0x02 for `RESPONSE TO INQUIRY`
- (16 bit) `Reference Number`:
  - `Nonce` for `Subtype` `EMERGENCY` and `CITIZEN REQUEST`
  - `Reference Number` of the `OFFICIAL ANNOUNCEMENT` for `Subtype` `RESPONSE TO INQUIRY` (with `OFFICIAL ANNOUNCEMENT` `subtypes` 0x31 and 0x32)
- (0 - (*max* - 19) bytes) `Content` (Unishox2-compressed text)
- (*tag*) AES-256-GCM AuthTag

When the message is sent, the whole `RDCP Payload` is replaced by the ciphertext corresponding to the plaintext `RDCP Payload` using authenticated encryption with the `static` `RDCP Header` fields (see below) as additional authenticated data. `Content` length is limited to respect that AES-256-GCM encryption adds a 16-byte AuthTag.

`CITIZEN REPORT` messages expect two `ACKNOWLEDGMENT` messages as response, one by the `Entry Point` (this is important for `MGs`, but can be handled internally if the `end-user` message was entered at a `DA`), and one by the active `HQ` device. When the `RDCP Infrastructure` is in `crisis mode`, these must be `positive (negative) acknowledgments`; when the `RDCP Infrastructure` is in `non-crisis mode`, at least the `Entry Point` must send a `negative acknowledgment` (there may be no `HQ` device online in this case).

If no `ACKNOWLEDGMENT` by the `Entry Point` is received within a certain timeframe, e.g., 60 seconds, of sending a `CITIZEN REPORT` message on a free channel, the message must be sent as a new message (i.e., with a new `Sequence Number`) again. If the `Entry Point` sends a `positive (negative) acknowledgment`, but no `ACKNOWLEDGMENT` by an `HQ` device is received within a larger reasonable timeframe, e.g., 15 minutes (depending, for example, on the LoRa spreading factor used in the scenario), the message must also be sent again as a new message (i.e., with a new `Sequence Number`). If the `Entry Point` sends a `negative acknowledgment`, the `end-user` must be informed appropriately. `End-users` should also be informed about received `positive (negative) acknowledgments` and ongoing retry attempts. Automated retry attempts should stop after 5 times (may be changed on a per-scenario basis) as either the device is not in range of the `RDCP Infrastructure` or the `RDCP Infrastructure` is malfunctioning. `End-users` may retry to send their message manually at a later point in time in such an unfortunate case.

NB: The timeouts used before re-sending a `CITIZEN REPORT` message with a new `Sequence Number` must consider the duration of propagation cycles for maximum-length `RDCP Messages` and can be adjusted on a per-scenario basis. Another maximum-sized `CITIZEN REPORT` and its corresponding `ACKNOWLEDGMENT` may already be on their way when a new `CITIZEN REPORT` is sent. Depending on the choice or LoRa parameters in the `RDCP infrastructure` (e.g., SF12 with 125 kHz bandwidth) and initial values for the `Retransmission Counter`, a full propagation cycle may take time in the order of 10 minutes. Re-sending a `CITIZEN REPORT` thus is a trade-off between ensuring that important lost messages are re-sent timely (as they may have very urgent content) and further straining an already busy LoRa channel. The current approach is to expect the `ACKNOWLEDGMENT` from the `Entry Point` quite soon but to wait for an `ACKNOWLEDGMENT` by the `HQ` longer, assuming that the `RDCP Infrastructure` will usually deliver the `CITIZEN REPORT` reliably and as soon as possible to the `HQ` once it hits the `Entry Point`, so the message can already be processed at the `HQ` even if it takes several minutes longer until the reporting citizen receives a confirmation.

It is recommended to allow for sending only a single `CITIZEN REPORT` per device for which the `ACKNOWLEDGMENT` by the `HQ` has not arrived yet. Sending multiple `CITIZEN REPORTs` in a row clogs the LoRa channel(s) so the corresponding `ACKNOWLEDGMENTs` might not get through, further increasing channel load due to re-sending the affected `CITIZEN REPORTs`.

### RDCP messages related to fetching older RDCP messages

RDCP devices might not be always powered on and therefore miss relevant messages. As a solution, those devices should have opportunities for "refuelling" without utilizing the available channel capacity too much.

For `MGs`, which are typically operated with portable power banks during a blackout-crisis and might be switched off to conserve power, e.g., while citizens in a household sleep, `DAs` periodically re-send `OFFICIAL ANNOUNCEMENTs` on their 868 MHz channel if it is currently free otherwise, with breaks of several seconds in-between. `DAs` only re-send `OAs` with a still valid `Message Lifetime` and limit the number of re-sent `OAs` as well as the `Retransmission Counter` for re-sent messages under duty cycle considerations. As a part of their `HEARTBEAT` messages, `MGs` report the latest `OA` they have received, and `DAs` can use this information to re-send only those messages that are missing on at least one `MG` in their vicinity.

While a `DA` ideally operates 24/7, especially during a real black-out crisis, its battery may run empty and as there is no solar power during nights, it will only start during day if there is enough light for solar power or when someone plugs in a new battery. When a `DA` reboots (also, e.g., after maintenance), it sends a `FETCH ALL NEW MESSAGES` message to a pre-configured neighbor `DA` on its 433 MHz LoRa channel. If this neighbor does not respond, the `DA` may fall back to pre-configured alternatives of other neighboring `DAs`. The `RDCP Payload` of a `FETCH ALL NEW MESSAGES` request is the 16-bit `Reference Number` of the latest `OFFICIAL ANNOUNCEMENT` message that the rebooted `DA` has stored/persisted, or 0x0000 if all currently relevant `OAs` should be sent (e.g., if the `DA's` hardware had to be replaced and its persistent storage was lost).

Alternatively, a `DA` may determine that it is missing only a specific `OA`, e.g., due to transmission errors. In this case, it sends a `FETCH MESSAGE` `RDCP Message` to its preconfigured neighbor where the `RDCP Payload` consists of the `Reference Number` of the missing message. Detecting missing `OAs` may, for example, be based on missing fragments of a multi-fragment `OA`, or if a non-zero lifetime update is received for a missing `OA`.

The neighboring `DA` responds by first re-sending the requested / all relevant `OFFICIAL ANNOUNCEMENTs` but may reduce the number of retransmissions (see `Retransmission Counter`) under duty cycle considerations and inhibits relaying by setting the `Entry Point` header field to 0xFF. Afterwards, the neighboring `DA` sends a `DELIVERY RECEIPT` message with no `RDCP Payload` to the requesting `DA` to signal that all available messages have been sent.

Neighboring `DAs` may rate-limit how often they respond to `FETCH` requests. For example, a `DA` with an almost empty battery may be in a "flapping" state, where it reboots, sends a `FETCH ALL NEW MESSAGES` message, loses power again and reboots every few minutes. Always responding to these requests may strain the 433 MHz channel significantly.

One rather specific problematic situation is that an `HQ` device received a `CITIZEN REPORT` message and already sent out an `ACKNOWLEDGMENT` for it, but then hardware-crashed before the message could be processed by a human, and therefore the message is lost. While this could be addressed by a store-and-fetch procedure between a (fresh) `HQ` device and `DAs`, the current solution is to have multiple `HQ` devices running in parallel but only one of them being active, i.e., sending messages, while the others listen only passively (and also receive the `CITIZEN REPORT` message, so it is not lost completely).

### RDCP Cryptographic Signature messages

`CRYPTOGRAPHIC SIGNATURE` messages carry data to authenticate `OFFICIAL ANNOUNCEMENT` messages if they are sent to the `RDCP broadcast address` or an `RDCP multicast address` for which no shared secret is established. While several other `RDCP Message Types` integrate cryptographic signatures into their `RDCP Payload`, `OAs` are typically too large to fit a cryptographic signature into the `RDCP Payload` without exceeding its maximum size.

The `RDCP Payload` of a `CRYPTOGRAPHIC SIGNATURE` message consists of:

- (16 bit) `Reference Number` of the `OA` to be authenticated
- (*sig*) Cryptographic signature

Details about cryptographic signature data are given below in the RDCP Message Authentication and Encryption section.

### Heartbeat messages

When the LoRa channel is otherwise free, `MGs` can optionally send `HEARTBEAT` messages periodically (e.g., every 30 minutes) to indicate that they are online. `HEARTBEAT` messages by `MGs` use the `HQ multicast address` as `Destination`, but are not relayed by `DAs`. `DAs` process these messages by recording the `Origin` and a timestamp. The `RDCP Payload` used by `MGs` consists of a 16-bit `OA` `Reference Number` (the latest persisted on the device) and the `RDCP address` of their current roaming recommendation `Entry Point`.

`DAs` can optionally send aggregated `HEARTBEAT` messages to the `HQ multicast address` periodically (e.g., every 30 minutes), which, unlike `MG` `HEARTBEATs`, are relayed and forwarded. The AES-256-GCM encrypted `RDCP Payload` consists of:

- (16 bit) Number of unique `MG` devices from which the `DA` has received at least one `HEARTBEAT` message since the last report.
- (0 - (*max*-2) bytes) `RDCP addresses` of those `MGs` (16 bit each). This list is currently truncated if there are too many relevant devices to fit the payload.
- (*tag*) AES-256-GCM AuthTag

### RDCP RTC messages

The `HQ` may send `RTCs` to specific RDCP devices with `RTC` support. The payload consists of:

- (8 bit) Delay in minutes to activate the `RTC`.
- (8 bit) Delay in multiples of 5 minutes when to restart, or 0 to disable restart.
- (8 bit) Indicator whether to persist the `RTC` (0x00 to disable, 0x01 to enable persistence).
- (n * 8 bit) `RTC`
- (*sig*) Cryptographic signature

### RDCP Routing-related messages

`DAs` require a `routing table` to know in which `timeslot` they are supposed to relay new `RDCP Messages` based on the `Entry Point` that started the `propagation cycle`. Internally, `DAs` store two versions of the `routing table`, the current (new) and the previous (old) one, so they can switch back to the previous version when requested by a `HQ` device, e.g., if the new version does not work as intended.

The `routing table` is also required to estimate when the `propagation cycle` for a specific `RDCP Message` will end, i.e., the channel will be free again, given that `propagation cycle lengths` may vary with `Entry Points`.

An `HQ` device sends AES-256-GCM encrypted `ROUTING INFORMATION` messages to individual `DAs`, given that `routing table` data for all `DAs` in an `RDCP Infrastructure` would not fit into a single `RDCP Message`. This also allows for selective updates of only a few `DAs'` `routing tables`, e.g. due to local changes in the overall mesh topology, without affecting the whole infrastructure; however, consistency must be ensured to avoid LoRa packet collisions depending on scenario-specific collision domains.

A `ROUTING INFORMATION` message's payload consists of:

- (16 bit) `Version number` of this `routing information`
- (8 bit) `Delay` in minutes before the new `routing table` becomes effective
- (n bytes) A list of `routing table commands`
- (*tag* bytes) AES-256-GCM AuthTag

`Routing table commands` are parsed byte-wise from left to right; a `command` may have arguments as subsequent bytes as listed below:

- 0xFF: Command to use the "current" `routing table` as the "previous" `routing table` and start with a clean "current" `routing table`.
- 0xFE: Command to use the "current" `routing table` as the "previous" `routing table` and keep the "current" `routing table` as-is.
- 0xFD: Command to edit the "current" `routing table` as-is without any impact on the "previous" `routing table`. (default)
- 0xEF: Use replacement-mode for subsequent entries, i.e., this `DA` may only relay in a single `timeslot` per `Entry Point`.
- 0xEE: Use append-mode for subsequent entries, i.e., this `DA` may relay in multiple `timeslots` per `Entry Point`. (default)
- 0xDF: Switch back to the "previous" `routing table`: The "current" `routing table` becomes the "previous" `routing table` and vice versa.
- 0xCF followed by 2-byte tuple (8 bits for `EP Identifier`, 8 bits for `propagation cycle length`): Clear relay settings for the `Entry Point` given by its `Identifier`, i.e., do not relay for this `Entry Point`; however, note the `propagation cycle length` when a `RDCP Message` enters the mesh network through the identified `Entry Point`.
- 0xCE followed by 2-byte tuple (8 bits for `EP Identifier`, 4 bits for `timeslot`, 4 bits for `propagation cycle length`): Depending on mode, replace the relaying setting for the `Entry Point` given by its `Identifier` or append to it. Relay new `RDCP Messages` initiated by this `Entry Point` in the given `timeslot` and note the given `propagation cycle length`.

Given the length of the AuthTag and the required 2 bytes per `Entry Point`, it might become necessary to send multiple `ROUTING INFORMATION` messages with the same `version number` to the same `DA`, e.g., one with the 0xFF and the following ones with the 0xFD commands. Note that each `DA` needs `routing table` entries for each `DA` (i.e., including itself) to know the individual `propagation cycle lengths`. Missing entries will be treated as follows: The message is not relayed in any `timeslot`, and a maximum-length `propagation cycle` is assumed.

The "new"/"current" `routing table` is started to be used at the end of the specified `delay`. If multiple `ROUTING INFORMATION` messages with the same `version number` are received, the `delay` specified in the last one is used. In general, newer `ROUTING INFORMATION` messages supersede previously received ones.

`DAs` respond to received `ROUTING INFORMATION` messages with an AES-256-GCM encrypted `ROUTING CONFIRMATION` message sent to the `HQ multicast address`. Its payload consists of:

- (16 bit) `Version number` of the "new" `routing table` that will be used
- (8 bit) `Delay` in minutes before the new `routing table` becomes effective
- (8 bit) Number of `Entry Points` for which data has been stored in the "new" `routing table` (accumulated over several `ROUTING INFORMATION` messages, if applicable)
- (8 bit) Number of `Entry Points` for which `RDCP Messages` will be relayed in at least one `timeslot`
- (*tag*) AES-256-GCM AuthTag

### RDCP Sequence Number handling messages

The `Sequence Numbers` used in the `RDCP Header` are sensitive in the context of Denial-of-Service (DoS) attacks, e.g., based on spoofed or replayed messages; in rare cases, improperly received `RDCP Messages` can also incidentally have a valid `Checksum` with a wrong `Sequence Number`.

Given that almost all `RDCP Messages` are spread throughout the whole `RDCP Infrastructure`, `HQ` devices can monitor the used `Sequence Numbers` and derive the real `Sequence Numbers` used by other devices from authenticated and encrypted messages: If the AES-256-GCM AuthTag is correct, the `Sequence Number` of the message it is contained in is authentic. This mechanism is used to correct `Sequence Number` inconsistencies to avoid problems with duplicate elimination.

If an RDCP device detects that an `RDCP Message` uses its own `RDCP Address` as `Origin` but a `Sequence Number` that has never been used by itself yet, it sends a `SEQUENCE NUMBER ALARM` to the `HQ multicast address` with the following payload:

- (24 bit) Seen inconsistent `Sequence Number`
- (*tag*) AES-256-GCM AuthTag

This message is sent with the next regular `Sequence Number` the device would use and must be propagated by `Relays` even if the `Sequence Number` indicates a duplicate. Rate limiting may be applied.

`SEQUENCE NUMBER ALERTs` are seen by all other devices in the topology and may trigger implementation-specific reactions. The affected device continues to use its regular `Sequence Numbers` as usual for the time being. `Entry Points` and `Relays` may also try to detect traffic and `Sequence Number` anomalies or conspicuous ways in which they receive new `RDCP Messages` and block/ignore sending devices appropriately.

A `HQ` device can send a cryptographically signed `SEQUENCE NUMBER OVERRIDE` message to set the last-seen `Sequence Number` for one or more devices. The `RDCP Payload` consists of:

- A list of 5-byte tuples consisting of (16-bit) `RDCP Address` and (24-bit) `Sequence Number` values, setting the new "most recently used" `Sequence Number` for the device with the given `RDCP Address`.
- (*sig* bytes) Cryptographic signature

The new `Sequence Numbers` become effective immediately after the `propagation cycle` of the `SEQUENCE NUMBER OVERRIDE` message. If the `HQ` itself becomes the victim of a spoofing attack, it might have to use a `RESET OF INFRASTRUCTURE` message first if no higher `Sequence Numbers` for its own messages remain available.

When receiving a `SEQUENCE NUMBER OVERRIDE` message, the directly affected device adjusts which next `Sequence Number` it is going to use and all other devices adjust their duplicate filtering tables accordingly.

Note that the combination of both `RDCP Message Types` is insufficient to efficiently handle persistent DoS attacks, e.g., those where spoofed `Sequence Numbers` are used over and over again. However, given the nature of scenarios and real-world deployments of `DA` devices in public places, there are much simpler ways to sabotage `RDCP Infrastructures`, such as jamming at least one of the two used LoRa channels or disabling a `DA`, e.g., by unplugging its power supply or removing its antenna cables. Properly preventing `Sequence Number` spoofing would require asymmetric cryptographic signatures verifiable by everyone for each message with a drastic impact on channel capacity and deployment complexity. Additional security measures may become part of future RDCP versions, along with tamper-proofing the used RDCP devices to prevent reading private keys from publicly deployed devices such as `DAs`. In the meantime, disabling the use of message types that carry neither a signature (or are accompanied by it) nor an AuthTag in production environments can be a workaround.

### RDCP Tunnel messages

RDCP can be used to tunnel other protocols over `RDCP Infrastructures`. For example, there may be sensors like weather stations attached to or in the vicinity of `DAs` that are designed to use LoRaWAN, but LoRaWAN infrastructures including multi-channel gateways might become unavailable during a crisis, e.g., due to power failures or inaccessibility of ISPs due to landline incidents. `DAs` that are able to receive relevant sensor data may encapsulate it in an RDCP `TUNNELED MESSAGE` and send it to the `HQ` multicast address or any other destination where it than can be fed into the usual processing backend (e.g., forward a LoRaWAN packet to TheThingsNetwork just like a LoRaWAN gateway for TTN does).

The payload of a `TUNNELED MESSAGE` consists of:

- (8 bit) Initial value of the `Retransmission Counter` header field to use at `Relays`. Depending on the importance of `tunneled data`, the `Origin` of such a message suggests this value. The `Entry Point` may enforce a lower value or apply other means of rate limiting to ensure sufficient channel capacity for other `RDCP Messages`. `Relays` participating in the `propagation cycle` must use the value set in this field by the `Entry Point`.
- (8 bit) `Tunneled data type`. Valid values are currently chosen on a per-scenario base. For example: 0x00 LoRaWAN packet, 0x01 IPv4 packet, 0x02 IPv6 packet, 0x03 locally attached sensor data. The `Destination` device will need a decoder or packet handler to further process the `tunneled data`.
- (n * 8 bit) The `tunneled data` itself.

`Tunneled data` is generally expected to fit into a single `RDCP Message`. If fragmentation becomes necessary, the `tunneled data type` and the structure of the `tunneled data` can be chosen accordingly to support multiple fragments while keeping `Relay` implementations agnostic of those `tunneled data type`-specific details.

## RDCP Relaying and Flooding

`RDCP Messages` with an `HQ` or `MG` device as `Origin` are first sent by the `Origin` device to be received by the header-designated `Entry Point` `DA` over the 868 MHz LoRa channel (independent of the `Destination` of the message). The `Entry Point` `DA` relays the message on the 433 MHz channel to start the `propagation cycle`. `RDCP Messages` originating at a `DA` implicitly have this `DA` as their `Entry Point`. Messages that should not be relayed use the magic value 0xFF in their `Entry Point` `RDCP Header` field.

Each `DA` has a `routing table` that indicates in which `timeslot(s)`, if any, it should relay `RDCP Messages` with the given `Entry Point`. The `routing table` is set up by a `HQ` device based on its overview of the overall network topology (i.e., available `DAs` and the strengths of the links between them, as reported by `DA STATUS RESPONSEs`) using `ROUTING INFORMATION` messages. The `propagation cycle` must be set up on a per-`Entry Point` basis and the order of individual `Relays` must obviously be chosen so that a `Relay` has a sufficiently high probability of successfully receiving the `RDCP Message` before its (first) `timeslot` for relaying it has come. Any `Relay` may be assigned to multiple `timeslots`, e.g., to avoid unused `timeslots` in the overall `propagation cycle` if equal length per `propagation cycle` is desired, or to maximise the chances that the `RDCP Message` is successfully flooded throughout the topology by increasing the number of transmissions for selected `Relays`. However, regionally applicable duty cycle constraints and overall channel capacity must be considered.

The number of `timeslots` used in a `propagation cycle` may vary based on the `Entry Point`. Typically, `propagation cycles` will be constructed in a way that each `DA` will receive the `RDCP Message` from at least two neighboring `Relays` and that single neighbors of otherwise isolated `DAs` will relay in at least one `timeslot`.

During the initial roll-out of a new `RDCP Infrastructure`, `DAs` must be provisioned with a default `routing table` to ensure that subsequent `ROUTING INFORMATION` messages reach all `DAs`.

Overall, the same message is sent by different `Senders` and each `Sender` (including `HQ` and `MG` devices) sends several `RDCP Message Types` multiple times according to the specified values for the `Retransmission Counter`. To ensure that each unique message is only fully processed once by each device, a `duplicate filtering` mechanism is used: `Properly received` (see below) `RDCP Messages` are checked for being already known based on their `Origin` and their `Sequence Number`. A message is only new if it uses a `Sequence Number` that is different from previous messages by the same `Origin` by incrementation (usually +1). Messages that are not new are usually not processed any further with certain exceptions detailed in the `RDCP message types` specification; for this reason, re-sending old `OAs` is an uncritical operation (e.g., when responding to a `FETCH ALL NEW MESSAGES` request) and, e.g., `CITIZEN REPORT` messages not acknowledged by an `HQ` device must be sent with a fresh `Sequence Number` in a retry attempt.

The use of `Sequence Numbers` along with static cryptographic key material makes the `RDCP Infrastructure` generally vulnerable to spoofing and replay attacks. However, given that different messages by the same `Origin` may have the same overall content (e.g., distinct `ECHO REQUEST` messages), a `Sequence Number` is necessary to distinguish between "old" and "new" messages. The problem is partially addressed as follows:

- The range for `Sequence Numbers` is large enough to avoid overflows for several years even for the most frequently sending RDCP devices (assuming regular operations with only a few regularily executed stress tests, e.g., during crisis exercises).
- `Sequence Number` inconsistencies, e.g., due to spoofed messages, can be detected by the `HQ` and handled with `SEQUENCE NUMBER ALARM` and `SEQUENCE NUMBER OVERRIDE` messages.
- The use of `RDCP message types` that do not use any kind of cryptographic authentication can be minimised or disabled in production environments.

An `RDCP Message` counts as `properly received` when it has a valid `Checksum` field in its `RDCP Header`. Messages with an invalid `Checksum` are treated as if they were never received, as any information contained in their `RDCP Header` and `RDCP Payload` is unreliable. Future RDCP versions may add additional means of forward error correction, also leveraging the re-transmissions of important `RDCP message types`.

When a `DA` properly receives a new message on its 433 MHz LoRa channel, it sends it on its 868 MHz LoRa channel (as soon as it is safely considered free) unless the `Destination` address is a `DA` (maybe even itself). Forwarding on the 868 MHz LoRa channel is independent of the `DA`'s role as designated `Relay` on the 433 MHz LoRa channel; for example, an `OA` should reach all `MGs` on the 868 MHz channel independent of whether their `DA` serves as a 433 MHz `Relay` for this particular `OA`. Messages received on the 868 MHz channel are also sent on the 868 MHz channel by the `Entry Point`, as the `Destination` may be in its vicinity and may receive messages sent by the `Entry Point`, but not by the other `MGs` such as the `Origin` in the same area.

New `RDCP Messages` must only be sent when the `propagation cycle` of a previous message has ended. Random delays and channel activity detection ("listen before talk") should additionally be applied before sending a new message, as other devices might also have new messages queued and wait for a free channel.

Collisions may still occur if new messages are sent roughly at the same time in different parts of the topology (the classical "hidden node problem" where "listen before talk" inherently fails). Two kinds of "collisions" must be considered:

- A "real" LoRa packet collision means that neither message is received properly by an affected `DA`. For this reason, `end-user` generated messages of `RDCP Message Type` `CITIZEN REPORT` are sent with a new `Sequence Number` if they are not confirmed by an `HQ` device via an `ACKNOWLEDGMENT` within a reasonable time. These 1st category collisions cannot be handled by `DAs` as they do not receive either of the involved messages properly and thus cannot be aware of the details of the problem.
- A `DA` receives a second `RDCP Message` before the `propagation cycle` of a first `RDCP Message` has finished. These 2nd category collisions can be handled by either of:
  - Postponing:
    - The `DA` handles the first received `RDCP Message` as it usually would (i.e., as `Relay` or simply as recipient).
    - Relaying the conflicting message uses the end of the first message's `propagation cycle` as scheduling baseline instead of the current point in time, i.e., the second `propagation cycle` is paused and resumed when the channel is free again.
  - Filtering:
    - If the receiving `DA` is a `Relay` for both messages, only one of them is relayed. `RDCP Message Types` are prioritized for this purpose, and `OFFICIAL ANNOUNCEMENTs` and `CRYPTOGRAPHIC SIGNATUREs` are chosen before `ACKNOWLEDGMENTs` and `CITIZEN REPORTs` (in this order). If no such prioritized message is affected, the one furthest in the `propagation cycle` is relayed.
    - If the receiving `DA` is a `Relay` for only one of the messages, it is *not* relayed. However, if the message to relay is an `OFFICIAL ANNOUNCEMENT` or `CRYPTOGRAPHIC SIGNATURE`, it is sent after the `propagation cycle` of the conflicting message has ended and a random delay is applied, just as if the affected `DA` was the `Entry Point` of the `OA`.
    - If the receiving `DA` is not a `Relay` for either message, the "collision" is ignored and both messages are processed as usual.
  - All `DAs` in an `RDCP Infrastructure` must implement the same strategy for handling 2nd category collisions.

The prioritized order of `RDCP Message Types` first reflects that `OAs` and their accompanying `CRYPTOGRAPHIC SIGNATUREs` are not sent again if they are lost on the way, and the `HQ` device has no way to detect this situation. Second, missing `ACKNOWLEDGMENTs` also cause the message to be confirmed having to be re-sent, further increasing the load on a currently strained LoRa channel. Third, `end-user` generated `CITIZEN REPORT` messages are important, but implicitly re-sent in case of loss. Finally, a complete loss of other `RDCP Messages Types` does not impact the core functionality of the `RDCP Infrastructure`, so it is acceptable in high-load situations.

`MGs` must support using a preconfigured `DA` as the `Entry Point` for their `RDCP Messages`, usually the geographically nearest or the one with the best RSSI/SNR and PDR values; it is set during the device provisioning process. If the primary `Entry Point` does not work (e.g., it does not send `ACKNOWLEDGMENTs` to `CITIZEN REPORTs`), the `MG` may fall back to preconfigured alternatives or use a `DA` from which it received `RDCP Messages` when the `DA` was the `Sender`. The latter case may also be the default for `MGs`, i.e., they can be roaming by using the `DA` as `Entry Point` that currently provides the best signal strength; this is referred to as `roaming recommendation` and reported in `MGs'` `HEARTBEATs`. Special-purpose `MGs` (i.e., those not for `end-users` and households, but for emergency services or maintenance personnel) may allow for a manual selection of which `DA` to use as `Entry Point` at run-time, although following the `roaming recommendation` is usually sufficient.

RDCP traffic on the 868 MHz channel does not use multiple subsequent `timeslots` but relies on channel activity detection. `MGs` are supposed to keep the channel free if important messages are expected following a received message (referred to as `corridor`). For example, if an `MG` receives a `CITIZEN REPORT` message by a neighboring `MG`, it should delay sending an own message for, e.g., 60 seconds or until the corresponding `ACKNOWLEDGMENT` by the `Entry Point` has been received. Similar, `HQ` devices should take the overall `propagation cycle` of their current message under consideration before sending the next one. While `Entry Points` typically implement small order-preserving buffers for outgoing messages, their storage size for this purpose is deliberately limited and strict `Origin`-based throttling plays a key role in keeping the channel free for high-priority messages and fairness of the overall system.

## RDCP Message Authentication and Encryption

RDCP devices are provisioned by an `RDCP Infrastructure Operator`, which allows for storing both individual and scenario-wide cryptographic key material on each device.

For authenticating `RDCP Messages` whose `RDCP Payload` includes a cryptographic signature field of length *sig*, asymmetric cryptography is used and the `Public Key` of each device that is supposed to create such cryptographic signatures is stored on all other devices. This primarily affects `HQ` devices, but also includes `MGs` for maintenance operations by authorized personnel.

Current prototype implementations use Schnorr signatures with *sig* = 65 bytes because of their "small" size.

For `RDCP Messages` that integrate a cryptographic signature, it is created using a SHA-256 hash of the `static RDCP Header fields`, i.e., `Origin`, `Sequence Number`, `Destination`, `Message Type`, and `Payload Length`, as well as the `RDCP Payload` excluding the signature itself. Other `RDCP Header` fields are not used for the hash calculation as they will change during the `propagation cycle` or by trying different `Entry Points`.

For `CRYPTOGRAPHIC SIGNATURE` messages accompanying `OFFICIAL ANNOUNCEMENT` messages, the signature is created for a SHA-256 hash of the `RDCP Header` fields `Origin`, `Destination`, and `Message Type` as well as the `Subtype`, the `Reference Number`, the `Lifetime`, and the `More Fragments` field of the first fragment along with the concatenated `Contents` of all fragments (*after* their Unishox2-compression).

Separate `SIGNATURE` messages may be sent ahead of or after their corresponding `OAs`. It is recommended that `end-user` devices clearly indicate that an `OA's` authenticity could not be verified yet in the latter case, and remove or tag `OAs` whose `SIGNATURE` was invalid. Implementations should also consider to remove `OAs` for which an accompanying `SIGNATURE` does not arrive within reasonable time.

For `CITIZEN REPORTs`, a shared secret 256-bit key specific for each non-`HQ` device (which is also stored on all `HQ` devices) is used for AES-256-GCM authenticated encryption using the `static RDCP Header` fields (see above) as *additional data* and the original `RDCP Payload` as *plaintext*, and then replacing the `RDCP Payload` with the resulting *ciphertext*. Note that the overall `RDCP Payload` size maximum must not be exceeded, also accounting for the 16-byte AES-256-GCM authentication tag. A unique IV is derived from the `static RDCP Header` fields per message.

For `OFFICIAL ANNOUNCEMENTs` that are sent to a unicast address, the same authenticated encryption procedure is used as for `CITIZEN REPORTs`. `End-user` devices must verify the authenticity of received `OAs` sent to their unicast `Destination` address first, and only then can process decrypted subheader fields such as `Subtype`.

## Duty Cycle and Legal Considerations

- RDCP is designed to be used over LoRa channels, although other underlying LPWAN and more generic network technologies could probably be used.
- The general assumption therefore is that radio bands are used for which no dedicated license or frequency allocation is required.
- However, such ISM / SRD bands typically have regional duty cycle regulations and/or other restrictions that must not be violated.
- In its `non-crisis mode`, RDCP (or, more precisely, its prototype implementations) is a very low-traffic protocol as only public event announcements and telemetry/management data are transmitted.
- In its `crisis mode`, where also `DAs` and `MGs` can be used to bring new messages into the mesh, traffic can increase arbitrarily in theory.
- We assume that in practice the traffic volume will still be rather low simply because we do not expect masses of `OFFICIAL ANNOUNCEMENTs` by public authorities or `CITIZEN REPORTs` by `end-users`, based on workshops held with authorities and citizens who experienced a real-world crisis in September, 2023.
- However, one part of the prototype deployment within the ROLORAN research project in the `Neuhaus Scenario` is to gather telemetry data that gives more detailed insight into real-world usage during disaster exercises and, eventually, real crises.
- Given the involvement of public authorities and emergency services, we will use these project results to determine whether dedicated frequencies or other special permits seem to be necessary or useful, and then update our recommendations for other RDCP `scenarios`.
- As a rule of thumb, RDCP enforces channel activity detection along with delays on the 868 MHz channel and uses a `timeslot`-based mechanism with initial channel activity detection on the 433 MHz channel. Given the number of devices involved in a typical scenario and, for example, the actual `routing tables` used, upper bounds for channel usage per device can be derived. It seems generally fair to assume that RDCP devices are "well-behaved" as long as they are properly configured.

## References and Links

- [ROLORAN Dual-Channel RDCP Relay software implementation](https://github.com//roloran/rdcp-relay)
- [ROLORAN LilyGo T-Deck (ROLODECK) MG software implementation](https://github.com/roloran/ROLODECK)
- [ROLORAN-Terminal software for interacting with RDCP devices](https://github.com/roloran/ROLORAN-Terminal)
- [ROLORAN Open Hardware for RDCP devices](https://github.com/roloran/ROLORAN-Hardware)
- [ROLORAN implementation of cryptographic Schnorr signatures](https://github.com/roloran/SchnorrSig)
- [ROLORAN project website (German)](https://www.unibw.de/software-security/forschung/roloran)
