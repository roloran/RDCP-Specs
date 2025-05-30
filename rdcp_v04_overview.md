# RDCP v0.4 Quick Overview

- RDCP is the ROLORAN Disaster Communication Protocol.
- This document covers v0.4 of RDCP as of mid-2025.
- RDCP is currently in development as part of the EU-funded (#NextGenerationEU) [dtec.bw research project ROLORAN](https://dtecbw.de/home/forschung/unibw-m/projekt-roloran) in cooperation with [Gemeinde Neuhaus, Austria](https://neuhaus.gv.at/).
- This document shortly summarizes the most important aspects of RDCP for implementers and interested third parties. More detailed information is currently available only in German and project-internal, but might be published later.

## RDCP Basics

- `RDCP Messages` are transmitted as the payload of LoRa packets.
- Each `RDCP Message` consists of an `RDCP Header` followed by an `RDCP Payload`.
- The `RDCP Header` has a fixed size of 16 bytes (in RDCP v0.4; it used to be 11 bytes earlier).
- The size of the `RDCP Payload` depends on the `RDCP Message Type` used and can vary between 0 (no payload) and 184 bytes (to not exceed an overall LoRa packet payload of 200 bytes).
- Selected `RDCP Message Types` use cryptographic signatures or authenticated encryption to enable the verification of the `Origin` of the `RDCP Message` and keep the payload content confidential if it may carry sensitive information.

## RDCP Infrastructure

- RDCP applies controlled flooding of messages by relaying in a two-tier mesh/star network topology.
- `RDCP Messages` are flooded in a mesh of `RDCP Infrastructure` backbone nodes referred to as "Digitale Anschlagtafeln" (`DAs`, digital bulletin boards), typically using one common 433 MHz LoRa channel.
- In the vicinity of each `DA` there may be mobile devices ("Mobile Geräte"/`MGs`, also referred to as "Pagers"). `DAs` use an 868 MHz LoRa channel to communicate with `MGs`.
- Each `RDCP Infrastructure` has headquarters (`HQ`, "Krisenstab"). `HQ` devices are operated by public authorities and emergency services and have additional, privileged functionality at their disposal, such as sending `Official Announcements` (`OAs`) to facilitate authorities' communication with citizens. Typically only one `HQ` device is active at any given time, others may be running in stand-by mode.
- An `RDCP Infrastructure` is the set of all RDCP-capable devices in one `Scenario`. `HQ` devices and `DAs` are managed by an `RDCP Infrastructure Operator`, which also prepares/configures and hands out `MGs` to `end-users` (citizens) in a provisioning/roll-out/deployment process.
- An `RDCP Infrastructure` operates in either `crisis mode` or `non-crisis mode`. In both modes, the `HQ` devices can send new `OAs` to `DAs` and `MGs`, but only in `crisis mode` the `DAs` and `MGs` can also be used to send `Citizen Reports` back to `HQ`.

## Neuhaus Scenario

- The first larger-scale real-world deployment of an `RDCP Infrastructure` is currently rolled out in the municipality of Neuhaus, Austria.
- In the `Neuhaus Scenario` for 2025, there are 10 `DAs` and around 100 `MGs` as constrained by project resources.
- `DAs` work as both mesh `Relays` and devices that can be used interactively (the latter is similar to the `MGs`).
- `MGs` are implemented using Lilygo T-Deck hardware with a custom-made firmware prototyped in the ROLORAN research project.
- `DAs` use custom-made hardware prototyped in the ROLORAN project with a focus on energy efficiency and autarchy for longer-term total black-out crisis scenarios.
- The `HQ` devices use custom-made software prototyped in the ROLORAN project running on COTS notebooks, using T-Deck hardware via USB as "LoRa modem" to connect to nearby `DAs`.
- RDCP can be used for other scenarios, but it is currently primarily developed for the `Neuhaus Scenario` with its specific requirements. Several RDCP design decisions and limitations are also influenced by what we can achieve in practice when prototyping all the hardware and software with limited resources during the research project ROLORAN.
- The limited maturity of any ROLORAN artifacts including specifications, software implementations and hardware prototypes reflects the status of a research project. Contributions by volunteers are welcome, as is cooperation and joint development interest by third parties.

## RDCP Addresses and DA Identifiers

- RDCP uses 16-bit addresses as a trade-off between compact headers and support for a sufficiently large number of devices in the overall `RDCP Infrastructure`.
- An individual `RDCP address` is assigned to each device.
- Certain `RDCP Messages`, such as `OAs`, can be sent to the `RDCP broadcast address`, i.e., everyone.
- `RDCP multicast addresses` are used for groups of devices.
- Each RDCP device 'listens' to its individual `RDCP address`, the `RDCP broadcast address`, and the `RDCP multicast addresses` of the device groups it is assigned to. `DAs` can serve as `Relays` for any other `RDCP Messages` as well.
- RDCP's 16-bit address space is currently used as follows:

  | RDCP address/range | Usage                          |
  | -----------------: | :----------------------------- |
  | 0x0000             | *reserved*                     |
  | 0x0001 - 0x00FE    | `HQ` devices                   |
  | 0x00FF             | `HQ` multicast address         |
  | 0x0100 - 0x01FF    | *reserved*                     |
  | 0x0200 - 0x02FF    | `DAs`                          |
  | 0x0300 - 0xAEFF    | `MGs` for citizens             |
  | 0xAF00 - 0xAFFF    | `MGs` for tests and exercises  |
  | 0xB000 - 0xBFFF    | Group/multicast addresses      |
  | 0xC000 - 0xFEFF    | `MG` s for special purposes    |
  | 0xFF00 - 0xFFFE    | *reserved*                     |
  | 0xFFFF             | Broadcast address              |

- For the relaying-related `RDCP Header` fields, which are explained below, additional 4-bit identifiers are assigned to `DAs`.
- The following `RDCP addresses` and `Relay Identifiers` have been assigned to `DAs` in the `Neuhaus Scenario` so far:

  | `DA` name/location | `RDCP address` | `Relay Identifier` |
  |--------------------|:--------------:|:------------------:|
  | Neuhaus            | 0x0200         | 0x0                |
  | Illmitzen summit   | 0x0201         | 0x1                |
  | Illmitzen valley   | 0x0202         | 0x2                |
  | Motschula          | 0x0203         | 0x3                |
  | Pudlach            | 0x0204         | 0x4                |
  | Schwabegg          | 0x0205         | 0x5                |
  | Heiligenstadt      | 0x0206         | 0x6                |
  | Wesnitzen          | 0x0207         | 0x7                |
  | Bach               | 0x0208         | 0x8                |
  | Berg ob Leifling   | 0x0209         | 0x9                |

## RDCP Header

- The `RDCP Header` has a fixed size of 16 bytes.
- An `RDCP Message` consists of an `RDCP Header` followed by an `RDCP Payload` (if there is any payload for the used `RDCP Message Type`; otherwise, the whole message consists of only the header).
- Important nomenclature:
  - The device transmitting an `RDCP Message` is referred to as `Sender`.
  - The device that created an `RDCP Message` and initially transmitted it is referred to as `Origin`.
  - The first `DA` to send a new message for distribution in the mesh network on the 433 MHz LoRa channel is referred to as `Entry Point`. `MGs` typically choose their geographically nearest `DA` as entry point but getting the message flooded in the mesh may fall back to alternatives if a specific `DA` (the primary `Entry Point`) is currently unavailable. `DAs` serve as their own `Entry Points` for locally created messages (e.g., through interactive `DA` use or when responding to incoming messages, such as telemetry requests).
  - Each `Sender` designates up to three `Relays` for the currently transmitted message according to rules laid out below. To avoid packet collisions, each designated `Relay` is also assigned a `Delay`.
  - The set of three `Relays` specified by the `Entry Point` is referred to as `First Hops`. `Relays` specified by `First Hops` are referred to as `Second Hops`. One of the `Second Hops` can then specify a `Third Hop`. We currently do not use more than three hops for flooding the mesh and have some restrictions in place to limit the number of `Relays` used in each step as specified below.
  - `RDCP Messages` can be transmitted multiple times by the same `Sender` to compensate for transmission errors. The `Retransmission Counter` in the `RDCP Header` indicates how often the same message will be sent again by the same `Sender` in the same `Timeslot` (see below). The number of retransmissions (i.e., the initial value of the `Retransmission Counter`) is specified for each `RDCP Message Type` and must be used by RDCP-compatible devices if any `Relay` is designated.
  - `Delays` are specified as number of `Timeslots`. The actual duration (in wallclock time) of one `Timeslot` is calculated as nrt * (airtime + 1000 ms), where *nrt* is the (initial) number of retransmissions for the `RDCP Message Type`, *airtime* is the LoRa packet time-on-air (which depends on the overall LoRa packet size and the LoRa parameters chosen in the `Scenario`, such as coding rate, spreading factor, preamble length, and bandwidth), and 1.000 ms are currently used as a buffer time between transmissions to cover processing times in less performant embedded systems.
  - The overall duration it takes for a message to be transmitted by its `Entry Point` and all hops is referred to as `propagation cycle duration`. `Entry Points` are supposed to delay sending a new message at least until the `propagation cycle` of the previous message has finished to avoid packet collisions and message reordering; see the section about RDCP Relaying and Flooding for details.
- The 16-byte `RDCP Header` consists of:

  | Header field name        | Field size in bits | Description                                                                         |
  | ------------------------ | :----------------: | ----------------------------------------------------------------------------------- |
  | `Sender`                 | 16                 | `RDCP address` of `Sender`                                                          |
  | `Origin`                 | 16                 | `RDCP address` of `Origin`                                                          |
  | `Sequence Number`        | 16                 | Message `Sequence Number`                                                           |
  | `Destination`            | 16                 | `RDCP address` of message recipient                                                 |
  | `Message Type`           |  8                 | `RDCP Message Type`                                                                 |
  | `Payload Length`         |  8                 | Size of `RDCP Payload` in bytes                                                     |
  | `Retransmission Counter` |  8                 | `Retransmission Counter` for currently transmitted message                          |
  | `Relay/Delay 1`          |  8                 | `Relay Identifier` and delay in `Timeslots` for first designated `Relay`            |
  | `Relay/Delay 2`          |  8                 | `Relay Identifier` and delay in `Timeslots` for second designated `Relay`           |
  | `Relay/Delay 3`          |  8                 | `Relay Identifier` and delay in `Timeslots` for third designated `Relay`            |
  | `Checksum`               | 16                 | CRC-16 (CCITT) checksum of all previous `RDCP Header` fields and the `RDCP Payload` |

- The `Sequence Number` is specific for the `Origin` of the `RDCP Message` and must be incremented by 1 for each new message, starting at 0 when the device is provisioned.
- The `Message Type` values, valid ranges for the `Payload Length`, and initial `Retransmission Counter` values for all `RDCP Message Types` are specified below.
- The `Checksum` is calculated for the whole `RDCP Message` (i.e., `RDCP Header` and `RDCP Payload`) except the 16-bit `Checksum` field itself.
- The three `Relay/Delay` header fields have a size of 8 bits each, but are logically split into two parts:
  - The upper (higher-valued) 4 bits are the `Relay Identifier`.
  - The lower (less significant) 4 bits give the number of `Timeslots` the designated `Relay` should wait before relaying the message.
  - The magic value of 0xEE is used to indicate that a `Relay/Delay` `RDCP Header` field is not used, i.e., no `Relay` is designated in this field.
  - The magic value of 0xFF is used to indicate that any `Relay` may transmit the message immediately when the channel is free, not caring about caused collisions due to a lack of coordination with potential other sending devices.
  - `MGs` use the `Relay/Delay 1` field to specify the `Entry Point` for their message with a `Delay` of 0x0 on the 868 MHz channel; the other two fields are set to 0xEE.
  - The magic value of 0xD is used as a `Relay Identifier` if a message must fill a `Relay/Delay` header field but assumes that no suitable `Relay` is available. Thus, 0xE as a `Relay Identifier` means "explicitly none", but 0xD means "I don't know".

## RDCP Message Types

This section specifies the `RDCP Message Types` currently in use.

### RDCP Message Types Overview

- In the following table, *sig* represents the size of a cryptographic signature (depending on implementation, see section RDCP Message Authentication and Encryption).

| Value | Name                      | Retransmissions | Payload size     | Payload content        | Short description                                 |
| ----: | :---                      | :-------------: | ---------------: | :--------------        | :----------                                       |
| 0x00  | `TEST`                    | 0               | 0 - 184 bytes    | plain text             | Reserved for test messages                        |
| 0x01  | `ECHO REQUEST`            | 0               | 0 bytes          | none                   | Simple unicast ("ping")                           |
| 0x02  | `ECHO RESPONSE`           | 0               | 0 bytes          | none                   | Response to echo request                          |
| 0x05  | `DA STATUS REQUEST`       | 0               | 1 byte           | binary data            | Telemetry data request for a `DA`                 |
| 0x06  | `DA STATUS RESPONSE`      | 0               | >= 9 bytes       | binary data            | Telemetry data from a `DA`                        |
| 0x09  | `BLOCK DEVICE ALERT`      | 0               | 4 bytes + *sig*  | binary data, signature | Exclude a device from relaying                    |
| 0x0A  | `TIMESTAMP`               | 0               | 6 bytes + *sig*  | binary data, signature | Date/time/mode information from an `HQ`           |
| 0x0B  | `DEVICE RESET`            | 0               | 2 bytes + *sig*  | `Nonce`, signature     | Reset instruction for a single device             |
| 0x0C  | `DEVICE REBOOT`           | 0               | 2 bytes + *sig*  | `Nonce`, signature     | Reboot instruction for a single device            |
| 0x0D  | `MAINTENANCE PREP`        | 0               | 2 bytes + *sig*  | `Nonce`, signature     | Switch single device to maintenance mode          |
| 0x0E  | `RESET OF INFRASTRUCTURE` | 2               | 2 bytes + *sig*  | `Nonce`, signature     | Reset of all participating devices                |
| 0x0F  | `ACKNOWLEDGMENT`          | 2               | 3 bytes + *sig*  | `Seq. Nr.`, signature  | Acknowledgment that message was received          |
| 0x10  | `OFFICIAL ANNOUNCEMENT`   | 4               | 6 - 184 bytes    | binary/Unishox2 data   | Official announcement by an `HQ`                  |
| 0x11  | `RESET OF OFF. ANN.`      | 2               | *sig*            | signature              | Revoke all previous `OFFICIAL ANNOUNCEMENTs`      |
| 0x1A  | `CITIZEN REPORT`          | 4               | 3 - 184 bytes    | binary/Unishox2 data   | Message from `end-user` to `HQ`                   |
| 0x20  | `FETCH ALL NEW MESSAGES`  | 0               | 2 bytes          | `Sequence Number`      | `DA` fetches all new messages from other `DA`     |
| 0x21  | `FETCH MESSAGE`           | 0               | 2 bytes          | `Reference Number`     | `DA` fetches specific message from other `DA`     |
| 0x2A  | `DELIVERY RECEIPT`        | 0               | 0 bytes          | none                   | Signal that response to 0x20/0x21 is completed    |
| 0x30  | `CRYPTOGRAPHIC SIGNATURE` | 4               | 2 bytes + *sig*  | binary data, signature | Separate signature for `OFFICIAL ANNOUNCEMENT`    |
| 0x31  | `HEARTBEAT`               | 0               | 4 - 184 bytes    | binary data            | Heartbeat / device registration                   |
| 0x32  | `RTC`                     | 0               | 4 - 184 bytes    | binary data, signature | `DA`-specific `RTC`                               |

### RDCP Test messages

RDCP `TEST` messages are *reserved* for testing and debugging purposes and not used during regular operation of `RDCP Infrastructures`. They are forwarded/distributed by `Relays` but not automatically interpreted by any device. The `RDCP Payload` consists of 0-184 bytes of plain text, i.e., no encoding, compression, or encryption is used.

### RDCP Echo Request and Echo Response messages

RDCP `ECHO REQUEST` and `ECHO RESPONSE` messages are similar to their ICMP equivalents in the Internet Protocol world. An RDCP-capable device is expected to respond with an `ECHO RESPONSE` message when it receives an `ECHO REQUEST` message. Both messages allow for a simple check of the connectivity/reachability of the target device, i.e., the `Destination` of the `ECHO REQUEST` and the `Origin` of the `ECHO RESPONSE`.

Both message types have an empty `RDCP Payload`. An `ECHO REQUEST` message may only be sent to a single target device (unicast). `ECHO REQUEST` messages sent to multicast/group addresses or the `RDCP broadcast address` are dropped (not forwarded) by `Relays` and should not be answered by receiving devices.

Sending an `ECHO REQUEST` usually is manually triggered using an `HQ` device or a dedicated management `MG` by the `RDCP Infrastructure Operator` to verify the target device's basic connectivity, e.g., when a `DA` appears to not be working properly. Software implementations for `DA` and `MG` `end-user` devices should not support sending `ECHO REQUESTs` through their primary user interfaces.

### RDCP DA Status Request and DA Status Response messages

RDCP `DA STATUS REQUEST` and `DA STATUS RESPONSE` messages handle `DA` telemetry data. An individual `DA STATUS REQUEST` message is typically sent to each `DA` once per hour by the active `HQ` device; fetching telemetry data should be temporarily omitted if the channel is known to be busy with other `RDCP Messages`.

The `DA STATUS REQUEST` should only be sent from an `HQ` device as `Origin`. It has a single-byte payload: 0x00 indicates that the `DA`'s telemetry counters should not be reset, while 0x01 indicates that the counters should be reset after preparing the `DA STATUS RESPONSE`.

`DA` devices are expected to respond with a `DA STATUS RESPONSE` message at least when the request appears to have come from an `HQ` device. The payload of response is binary data listing the `DA`'s telemetry counters, currently:

- (8 bit) Counter reset status: 0x00 if counters were not reset, 0x01 if counters were reset as a result of the request
- (16 bit) Battery status
- (16 bit) Number of unique received `RDCP Messages`
- (16 bit) Number of unique relayed/forwarded `RDCP Messages`
- (16 bit) Number of different `MG` devices from which messages were received
- n * (32 bit) List of other `DA` devices that were received by this `DA` when they were the `Sender` of an `RDCP Message`: (16 bit) Other `DA`'s `RDCP Address`, (8 bit) RSSI value, (8 bit) SNR value. The RSSI and SNR values are implementation-specific and can be taken from the most recently received message by the other `DA` or be an average value.

The counter reset instruction in the `DA STATUS REQUEST` message is only honored if it was sent by an `HQ` address. `DA STATUS RESPONSE` messages are always addressed to the `HQ multicast address` as `Destination`, not the `Origin` of the request.

### RDCP Device blocking message

 `HQ` devices can use `BLOCK DEVICE ALERT` messages to instruct `Relays` to drop (not forward) `RDCP Messages` originating from a specific device. It is intended for situations in which a `DA` or `MG` is clearly misused, e.g., to submit a larger amount of fake `CITIZEN REPORTs`.

The payload consists of:

- (16 bit) `RDCP Address` of the blocked device
- (16 bit) Blocking duration in minutes. The magic value of 0x0000 indicates that the device shall not longer be blocked.
- (*sig*) Cryptographic signature (message must be sent by an `HQ` device)

Obviously, `DAs` should only be blocked in extreme situations as they might be used by different persons.

### RDCP messages related to Device and Infrastructure Maintenance

`DA` devices are operated 24/7/365 and eventually require maintenance operations due to software updates, insufficient solar energy and power failures, or software-side deadlocks.

The `RDCP Messages` in this category prepare a single device, typically a `DA`, or the RDCP infrastructure part consisting of all `DAs`, for these maintenance operations. These messages may only be sent by an `HQ` or dedicated maintenance devices, as verified via cryptographic signatures.

An RDCP `DEVICE RESET` message instructs a single device, usually a `DA`, to perform a software-side reset of itself. This typically includes, among other actions, the deletion of volatile and selected persisted data, such as displayed `Official Announcements`, `RDCP Message` duplicate tables, and the queue of scheduled `RDCP Messages` to be sent. However, other persisted data, such as the last used own `Sequence Number` and own `RDCP Address`, must not be wiped. It is left to device implementation how this operation is carried out in detail. The `RDCP Payload` of a `DEVICE RESET` message consists of:

- (16 bit) `Nonce`
- (*sig*) Cryptographic signature

The `Nonce` (a number used only once) must be increased by 1 for each message of this type and avoids replay attacks because messages of this type are not checked for duplicates based on the `Sequence Number` (wrong `Sequence Numbers` are one of the problems to be solved with device resets). The target device therefore must also persist the most recently used value of this `Nonce` (in combination with the `Origin`).

An RDCP `DEVICE REBOOT` message is similar to a `DEVICE RESET`, but expects the target device to completely restart, including an operating system reboot (if applicable), and re-initialize all hardware components. The `RDCP Payload` consists of a `Nonce` and a cryptographic signature as above.

An RDCP `RESET OF INFRASTRUCTURE` message is also similar to a `DEVICE RESET`, but affects all devices that receive it, not only a single one, i.e., the `Destination` usually is the broadcast address. In addition, receiving devices reset their own used `Sequence Number`. However, `Relays` still have to forward the message before resetting themselves if they are able to do so. The `RDCP Payload` consists of a `Nonce` and a cryptographic signature as above. Messages of this type are expected to reach all `DAs`, but the majority of `MG` devices should be considered to be offline. It is sort of an operation "of last resort" if there was a major incident with the technical infrastructure that cannot be recovered from device-by-device.

Finally, an RDCP `MAINTENANCE PREPARATION` message turns on the maintenance mode for a single target device. In maintenance mode, the device can enable additional interfaces, e.g., for Ethernet, WiFi, or Bluetooth access, or over-the-air-updates. For `DAs`, this mode is usually activated in combination with physical access to the device. Such `RDCP Messages` may be sent by an `HQ` or dedicated maintenance `MG` devices and their `RDCP Payload` consists of a `Nonce` and cryptographic signature, similar to `DEVICE RESET`, `DEVICE REBOOT`, and `RESET INFRASTRUCTURE`. Maintenance always ends with a reboot of the target device to manually verify that it properly starts up also in the case of a temporary power failure. If maintenance mode has been activated accidentally, it can remotely be ended with a `DEVICE REBOOT` message. Support for `MAINTENANCE PREPARATION` messages is optional; devices may alternatively provide, for example, a physical switch for this purpose or temporarily enable maintenance access interfaces after a reboot.

Implementations of how messages in this maintenance category are handled by the target device(s) are expected to treat them with priority, in the sense that they still should work even if "nothing else works anymore". If they fail, physical access to the devices is required to, for example, power-cycle them.

### RDCP Timestamp and Mode Information messages

Currently, `DAs` and `MGs` do not have battery-buffered real-time clocks. They can be powered off for an arbitrary amount of time (e.g., during the night when a `DA`'s battery is empty, until there is enough solar energy during day again) and therefore have no knowledge about the real-world wallclock time. RDCP primarily addresses this issue by using durations, e.g., the `Message Lifetime` of an `Official Announcement`, instead of absolute start and end dates and times, despite the limitations of this approach: For example, if an `Official Announcement` should be displayed for five days, and a `DA` reboots on the third day and subsequently fetches the `OA` again, it will display it more than two days longer than intended.

RDCP `TIMESTAMP` messages can be sent, likely periodically, by an `HQ` device to distribute the current date and time in the `RDCP Infrastructure`. It is not intended as a time synchronization protocol and given the time it takes to propagate `RDCP Messages` via the `Relays`, no noteworthy accuracy is achieved. However, sometimes it can be useful to have absolute timestamps in internal logfile or display the current time to `end-users` of a device, even if it is off by a minute.

Similar to telemetry requests, timestamps should not be sent when the channel is known to be busy otherwise.

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

RDCP `ACKNOWLEDGMENT` messages confirm that another (the confirmed) `RDCP Message` has been successfully received by the `Origin` of the `ACKNOWLEDGMENT`. It is used only in the context of messages created by `end-users` (`RDCP Message Type` `CITIZEN REPORT`) and only acknowledges that the confirmed message was received on the technical communication level, not that it has been read and processed by a human (yet).

`ACKNOWLEDGMENT` messages have the `Origin` of the confirmed message as `Destination` if it is a unicast address and the following `RDCP Payload`:

- (16 bit) `Sequence Number` of the acknowledged message
- (8 bit) `Type of acknowledgment`:
  - Value 0x00: `Positive acknowledgment` - the acknowledged message was technically received and will be processed further.
  - Value 0x01: `Negative acknowledgment` - the acknowledged message was technically received, but cannot be processed further. This is intended for situations in which `end-users` send a `CITIZEN REPORT` message, but the overall `RDCP Infrastructure` is not in `crisis mode` and therefore noone at an `HQ` device will be able to quickly react manually. Official `DA` and `MG` devices disable the sending of such messages when they are certain to be outside of an crisis situation, but depending on which messages those devices do and do not receive or when they are powered on, they may consider themselves in `crisis mode` although the rest of the `RDCP Infrastructure` is not.
  - Value 0x02: `Positive negative acknowledgment` - the acknowledged message was technically received and will be processed further, but the `HQ` is currently unmanned.
- (*sig*) Cryptographic signature

The active `HQ` device and the `Entry Point` **must** respond to incoming `CITIZEN REPORT` messages with an `ACKNOWLEDGMENT`. The cryptographic signature is only mandatory for `HQ` devices currently, an `Entry Point` may omit it from the acknowledgment message. Messages to be acknowledged must have a unicast address as their `Origin`. Messages that have other `Origin` addresses are not acknowledged.

### RDCP messages related to Official Announcements

In `RDCP Infrastructures`, `Official Announcements` are text-based information prepared by a public authority or emergency service to be displayed on `DA` and `MG` devices. An `OFFICIAL ANNOUNCEMENT` message must be sent from an `HQ` device and can be addressed to either of:

- Everyone: The `Destination` address is set to the `RDCP broadcast address`.
- A group of devices: The `Destination` address is set to the corresponding `RDCP multicast address`.
- A single device: The `Destination` address is set to the target device's `RDCP Address`.

The payload of an `OFFICIAL ANNOUNCEMENT` (`OA`) consists of:

- (8 bit) `Subtype` of the `OA` message:
  - 0x10: `Non-crisis mode` announcement, for example information about a public event outside of a crisis situation
  - 0x20: `Crisis mode` announcement, for example information on the current situation and ongoing aid efforts
  - 0x22: Message lifetime update, used to change the lifetime of a previously sent `OA`
  - 0x30: Feedback to a `CITIZEN REPORT`, e.g., the information that help for a life-threatening emergency is on the way
  - 0x31: Further inquiry about a `CITIZEN REPORT`: Same as 0x30, but the recipient is expected to answer with another `CITIZEN REPORT`
- (16 bit) `Reference Number`:
  - For subtype 0x10 and 0x20 `OAs`, the `Reference Number` is a fresh `Nonce` that can be used to refer back to this `OA` later
  - For subtype 0x22 `OAs`, the `Reference Number` identifies the previously sent `OA` whose lifetime should be changed
  - For subtype 0x30 and 0x31 `OAs`, the `Reference Number` is the `Reference Number` of the related `CITIZEN REPORT`
- (16 bit) `Message Lifetime`; the initial lifetime for a new `OA` or the new lifetime for a previously sent `OA`:
  - A lifetime of 0 indicates that the `OA` should be deleted/invalidated. Only used with subtype 0x22 `OAs`.
  - Values in the range between 1 and 60000 specify the lifetime in minutes.
  - Values in the range between 60001 and 65534 specify the lifetime in full days (24h). For example, 60002 means 2 days, 60010 means 10 days.
  - A lifetime of 65535 (largest possible value in the 16-bit range) indicates infinite lifetime. Such messages can be explicitly deleted with a subtype 0x22 `OA`.
- (8 bit) `More Fragments` (see below, only relevant for `OAs` sent to the `RDCP broadcast address`)
- (0 - 162/178 bytes) `Content` of a new `OA` (see below) or (*sig*) Cryptographic signature for a `Subtype` 0x22 `OA`

`OAs` may be short, but typically contain longer text to keep the citizens well-informed with the necessary level of detail. For this reason, `OFFICIAL ANNOUNCEMENT` messages have the following specific characteristics:

- The textual `Content` of an `OA` is compressed/encoded using Unishox2, an algorithm dedicated to compress and decompress short texts. It should include a human-readable timestamp of when the `OA` was sent by the `HQ` device.
- If the Unishox2-compressed text does not fit into a single `OA` message (max. 162/178 bytes), the original announcement text is split into the necessary number of parts. These parts are then referred to as "fragments" and each fragment is sent in an `OFFICIAL ANNOUNCEMENT` message of its own. All these fragments share the same `Reference Number`, and the recipient must use the `More Fragments` value to bring the fragments into the correct order. For example, if an `OA` is split into three fragments, the first fragment has a `More Fragments` value of 2, the second fragment has a `More Fragments` value of 1, and the final fragment has a `More Fragments` value of 0. Fragments must also be sent in this order, so the recipient knows from the beginning whether more fragments are to be expected and can also identify whether fragments are missing until the end of the assembled `Content`. Multiple fragments can be used only for `OAs` sent to the `RDCP broadcast address`; `OAs` directed at single devices or multicast groups must always fit into a single `RDCP Message`. Note that each fragment must be properly Unishox2-decoded on its own; thus, when splitting a longer text into fragments, the original (uncompressed) text must be split so that its Unishox2-compressed representation does not exceed the size limit for each fragment.
- `OFFICIAL ANNOUNCEMENT` messages to the `RDCP broadcast address` are accompanied by separate `CRYPTOGRAPHIC SIGNATURE` message to verify their `Origin`. For multi-fragment `OAs` only a single `CRYPTOGRAPHIC SIGNATURE` message is used (detailed below in the section about `CRYPTOGRAPHIC SIGNATURE` messages). The `CRYPTOGRAPHIC SIGNATURE` message may be sent before or after the `OA` (or all of its fragments). `DAs` and `MGs` should appropriately inform `end-users` about `OAs` if a cryptographic signature is still missing or its verification failed.
- If the `HQ` device has a shared secret (cryptographic key material) established with the `Destination` device (unicast) or group (multicast), the complete `RDCP Payload` is encrypted and authenticated after the `Content` is Unishox2-compressed, with the s`tatic` `RDCP Header` fields as additional authenticated data (see below). In this case, there is no accompanying `CRYPTOGRAPHIC SIGNATURE` message, but the 16-byte AES GCM AuthTag must be considered regarding maximum message length.

Implementations must ensure by proper splitting into fragments that the maximum `Content` length is not exceeded.

An `RESET OF OFFICIAL ANNOUNCEMENT` message may be sent by an `HQ` device to trigger the deletion of all currently displayed/stored `OAs` on `DAs` and `MGs`. It is typically used at the end of crisis exercises or real crises to start fresh without removing all the other data that is deleted via the maintenance-category `RDCP Messages`. The `RDCP Payload` consists only of a cryptographic signature.

### RDCP messages for citizen reports

Only during a crisis, `DAs` and `MGs` as `end-user` devices can be used to send messages to the `HQ multicast address`. Either the most recently received `OFFICIAL ANNOUNCEMENT` along with its `Subtype` or the most recently received `TIMESTAMP` is typically used to derive whether the `RDCP Infrastructure` is in `crisis mode` (`OA` `Subtype` 0x10) or `non-crisis mode` (`OA` `Subtype` 0x20). However, one practical consideration is to always start `end-user` devices in `crisis mode` so that urgent emergencies can be reported immediately after powering the devices on without having to wait for a new `OA` or a periodic retransmission of recent `OAs`. It thus cannot be reliably avoided that `CITIZEN REPORTs` are also transmitted in `non-crisis mode`.

The user interfaces for `end-user` devices distinguish between three types of `CITIZEN REPORTs`, and they typically use different forms that have to be filled out with different data.

`Subtype` `EMERGENCY` messages are for reporting urgent, severe events involving life-threatening casualties that require assistance of highest priority to save lifes, such as medivac of a person in a hard-to-reach crisis area. `End-users` have to fill in the typical emergency report questions of "who reports?", "where is the emergency?", "what happened?" and "how many persons are affected?". The `Content` of the `RDCP Payload` of a `Subtype` `EMERGENCY` message concatenates the answers to these questions separated by the `#` symbol and is Unishox2-compressed.

`Subtype` `CITIZEN REQUEST` messages are used for still important, yet slightly less urgent reports by end users. The spectrum of potential reports is broad, ranging from observations of damaged public infrastructure (such as destroyed roads) and private property (damaged houses and flooded cellars) to a request to have groceries delivered somehow because households in a crisis area are running out of food and fresh water during a longer crisis. `End-users` typically have to answer "who reports?", select a type of request or observation from a list, and can enter a short text (about 100-150 characters) to describe their situation. Similar to above, the `Content` of the `RDCP Payload` of `Subtype` `CITIZEN REQUEST` messages concatenates this input and Unishox2-compresses it.

`Subtype` `RESPONSE TO INQUIRY` messages can only be sent if the `end-user` device previously received an `OFFICIAL ANNOUCEMENT` of `Subtype` 0x31. In such situations, the personnel at the active `HQ` device needs further information about a previous `CITIZEN REPORT` and `end-users` can fill in a free-text field in response. This `Content` is also Unishox2-compressed.

`CITIZEN REPORT` `RDCP Messages` always have the `HQ multicast address` as their `Destination`.

The plain-text `RDCP Payload` of `CITIZEN REPORT` messages consists of:

- (8 bit) `Subtype`:
  - 0x00 for `EMERGENCY`
  - 0x01 for `CITIZEN REQUEST`
  - 0x02 for `RESPONSE TO INQUIRY`
- (16 bit) `Reference Number`:
  - `Nonce` for `Subtype` `EMERGENCY` and `CITIZEN REQUEST`
  - `Reference Number` of the previous `CITIZEN REPORT` for `Subtype` `RESPONSE TO INQUIRY`
- (0-165 bytes) `Content` (Unishox2-compressed text)

When the message is sent, the whole `RDCP Payload` is replaced by the ciphertext corresponding to the plaintext `RDCP Payload` using authenticated encryption with the `static` `RDCP Header` fields (see below) as additional authenticated data. `Content` length is limited to 165 bytes as AES-256-GCM encryption adds a 16-byte AuthTag.

`CITIZEN REPORT` messages expect two `ACKNOWLEDGMENT` messages as response, one by the `Entry Point` (this is important for `MGs`, but can be handled internally if the `end-user` message was entered at a `DA`), and one by the active `HQ` device. When the `RDCP Infrastructure` is in `crisis mode`, these must be `positive (negative) acknowledgments`; when the `RDCP Infrastructure` is in `non-crisis mode`, at least the `Entry Point` must send a `negative acknowledgment` (there may be no `HQ` device online in this case).

If no `ACKNOWLEDGMENT` by the `Entry Point` is received within 60 seconds of sending a `CITIZEN REPORT` message on a free channel, the message must be sent as a new message (i.e., with a new `Sequence Number`) again. If the `Entry Point` sends a `positive (negative) acknowledgment`, but no `ACKNOWLEDGMENT` by an `HQ` device is received within 15 minutes, the message must also be sent again as a new message (i.e., with a new `Sequence Number`). If the `Entry Point` sends a `negative acknowledgment`, the `end-user` must be informed appropriately. `End-users` should also be informed about received `positive (negative) acknowledgments` and ongoing retry attempts. Automated retry attempts should stop after 5 times (i.e., roughly 90 minutes), as either the device is not in range of the `RDCP Infrastructure` or the `RDCP Infrastructure` is malfunctioning. `End-users` may retry to send their message manually at a later point in time in such an unfortunate case.

The timeouts used before re-sending a `CITIZEN REPORT` message with a new `Sequence Number` must consider the duration of propagation cycles for maximum-length RDCP messages. `CITIZEN REPORT` messages are currently sent 5 times per timeslot and there are 9 timeslots in the propagation cycle. Another maximum-sized `CITIZEN REPORT` and its corresponding `ACKNOWLEDGMENT` may already be on their way when a new `CITIZEN REPORT` is sent. Depending on the choice or LoRa parameters in the RDCP infrastructure (e.g., SF12 with 125 kHz bandwidth), a full propagation cycle may take time in the order of 10 minutes. Re-sending a `CITIZEN REPORT` thus is a trade-off between ensuring that important lost messages are re-sent timely (as they may have very urgent content) and further straining an already busy LoRa channel. The current approach is to expect the `ACKNOWLEDGMENT` from the `Entry Point` quite soon but to wait for an `ACKNOWLEDGMENT` by the `HQ` longer, assuming that the RDCP infrastructure will usually deliver the `CITIZEN REPORT` reliably and as soon as possible to the `HQ` once it hits the `Entry Point`, so the message can already be processed at the `HQ` even if it takes several minutes longer until the reporting citizen receives a confirmation.

### RDCP messages related to fetching older RDCP messages

RDCP devices might not be always powered on and therefore miss relevant messages. As a solution, those devices should have opportunities for "refuelling" without utilizing the available channel capacity too much.

For `MGs`, which are typically operated with portable power banks and might be switched off to conserve power, e.g., while citizens in a household sleep, `DAs` periodically re-send `OFFICIAL ANNOUNCEMENTs` on their 868 MHz channel if it is currently free otherwise, with breaks of several seconds in-between. `DAs` only re-send `OAs` with a still valid `Message Lifetime` and limit the number of re-sent `OAs` as well as the `Retransmission Counter` for re-sent messages under duty cycle considerations.

While a `DA` ideally operates 24/7, especially during a real black-out crisis, its battery may be empty and there is no solar power during nights. When a `DA` reboots (also, e.g., after maintenance), it sends a `FETCH ALL NEW MESSAGES` message to a pre-configured neighbor `DA` on its 433 MHz LoRa channel. If this neighbor does not respond, the `DA` may fall back to pre-configured alternatives. The `RDCP Payload` of a `FETCH ALL NEW MESSAGES` request is the 16-bit `Reference Number` of the latest `OFFICIAL ANNOUNCEMENT` message that the rebooted `DA` has stored/persisted, or 0x0000 if all currently relevant `OAs` should be sent.

Alternatively, a `DA` may determine that it is missing only a specific `OA`, e.g., due to transmission errors. In this case, it sends a `FETCH MESSAGE` `RDCP Message` to its preconfigured neighbor where the `RDCP Payload` consists of the `Reference Number` of the missing message.

The neighboring `DA` responds by first re-sending the requested / all relevant `OFFICIAL ANNOUNCEMENTs` but may reduce the number of retransmissions (see `Retransmission Counter`) under duty cycle considerations and does not designate any `Relays`. Afterwards, the neighboring `DA` sends a `DELIVERY RECEIPT` message with no `RDCP Payload` to the requesting `DA` to signal that all available messages have been sent again.

One rather specific problem is that an `HQ` device received a `CITIZEN REPORT` message and already sent out an `ACKNOWLEDGMENT` for it, but then hardware-crashes before the message can be processed by a human, and therefore the message is lost. While this could be addressed by a store-and-fetch procedure between a (fresh) `HQ` device and `DAs`, the current solution is to have multiple `HQ` devices running in parallel but only one of them being active, i.e., sending messages, while the others listen only passively (and also receive the `CITIZEN REPORT` message, so it is not lost completely).

### RDCP Cryptographic Signature messages

`CRYPTOGRAPHIC SIGNATURE` messages carry data to authenticate `OFFICIAL ANNOUNCEMENT` messages if they are sent to the `RDCP broadcast address` or an `RDCP multicast address`. While several other `RDCP Message Types` integrate cryptographic signatures into their `RDCP Payload`, `OAs` are typically too large to fit a cryptographic signature into the `RDCP Payload` without exceeding its maximum size.

The `RDCP Payload` of a `CRYPTOGRAPHIC SIGNATURE` message consists of:

- (16 bit) `Reference Number` of the `OA` to be authenticated
- (*sig*) Cryptographic signature

Details about cryptographic signature data are given below in the RDCP Message Authentication and Encryption section.

### Heartbeat messages

When the LoRa channel is otherwise free, `MGs` can optionally send `HEARTBEAT` messages periodically (e.g., every 30 minutes) to indicate that they are online. `HEARTBEAT` messages by `MGs` use the `HQ multicast address` as `Destination`, but are not relayed by `DAs`. They also use `0x0000` as `Sequence Number`, bypassing duplicate checks performed by `DAs`. `DAs` process these messages by recording the `Origin` and a timestamp. The `RDCP Payload` used by `MGs` consists of a 16-bit `OA` `Reference Number` (the latest persisted on the device) and the `RDCP address` of their current roaming recommendation `Entry Point`.

`DAs` can optionally send aggregated `HEARTBEAT` messages to the `HQ multicast address` periodically (e.g., every 30 minutes), which are relayed and contain a valid `Sequence Number`. The `RDCP Payload` consists of:

- (16 bit) Number of unique `MG` devices from which the `DA` has received at least one `HEARTBEAT` message since the last report.
- (0-182 bytes) `RDCP addresses` of those `MGs` (16 bit each). This list is currently truncated if there are more than 91 relevant devices.

## RDCP Relaying and Flooding

`RDCP Messages` with an `HQ` or `MG` device as `Origin` are first sent to an `Entry Point` `DA` over the 868 MHz LoRa channel. The `Entry Point` `DA` relays the message over the 433 MHz channel. At least one other `DA` is expected to receive the message. In the `RDCP Header` used by the `Entry Point`, exactly three other `DAs` are designated as `First Hops`. The magic value of 0xD is used as `Relay Identifier` if the `Entry Point` does not know a suitable `Relay`, e.g., because it only has one or two neighbors. Each first hop `Relay` sends the message after delaying for the assigned number of `Timeslots`. The first two of the three `First Hops` designate exactly two `DAs` as `Second Hops` (also eventually using 0xD as `Relay Identifier`). The first of the four `Second Hops` must then in turn specify one `Third Hop`. A `propagation cycle` ends when all `Relays` have finished sending the `RDCP Message`, i.e., after a total of 9 timeslots (1 timeslot for `Entry Point`, 3 timeslots for `First Hops`, 4 timeslots for `Second Hops`, 1 timeslot for `Third Hop`).

Therefore, the same message is sent multiple times by different `Senders` and each `Sender` (including `HQ` and `MG` devices) sends several `RDCP Message Types` multiple times according to the specified values for the `Retransmission Counter`. To ensure that each unique message is only fully processed once by each device, a `duplicate filtering` mechanism is used: `Properly received` `RDCP Messages` are checked for being already known based on their `Origin` and their `Sequence Number`. A message is only new if it uses a higher `Sequence Number` than any previous message by the same `Origin`. Messages that are not new are usually not processed any further; for this reason, re-sending old `OAs` is an uncritical operation (e.g., when responding to a `FETCH ALL NEW MESSAGES` request) and, e.g., `CITIZEN REPORT` messages not acknowledged by an `HQ` device must be sent with a fresh `Sequence Number` in a retry attempt. As an exception, `HEARTBEAT` messages sent by `MGs` use a fixed `Sequence Number` of `0x0000` but still are processed `DA`-internally; the intention here is to avoid `Sequence Number` overflows for `MGs`, which cannot be reset as easily as for the rest of the `RDCP Infrastructure`.

An `RDCP Message` counts as `properly received` when it has a valid `Checksum` field in its `RDCP Header`. Messages with an invalid `Checksum` are treated as if they were never received, as any information contained in their `RDCP Header` and `RDCP Payload` is unreliable.

When a `DA` properly receives a new message on its 433 MHz LoRa channel, it sends it on its 868 MHz LoRa channel (when it is free) unless the `Destination` address is a `DA` (maybe even itself). Forwarding on the 868 MHz LoRa channel is independent of the `DA`'s role as designated `Relay` on the 433 MHz LoRa channel; for example, an `OA` should reach all `MGs` on the 868 MHz channel independent of whether their `DA` serves as a 433 MHz `Relay` for this particular `OA`.

`DAs` are, as a part of their configuration, sufficiently aware of the network topology of their part of the overall `RDCP Infrastructure` and thus designate three other `DAs` as `First Hop` `Relays` when they are the `Entry Point`; the first two of the three `First Hops` designate two `Second Hops` each. The first of those four `Second Hops` must designate one `Third Hop` `Relay`.

Usually, those neighboring `DAs` are designated as `Relays` to which a good packet delivery rate (PDR) is known and which can help to flood new messages throughout the network topology most efficiently (i.e., not necessarily the nearest neighbors but rather far away while still maintaining a reliable PDR). `DAs` may adjust their choice of designated `Relays` at run-time or based on the `Sender` or type of an `RDCP Message`, but any optimizations should be applied conservatively. For example, if a specific other `DA` is the `Entry Point` or a previous hop of a certain message and the current `DA` 'hears' it sending the message before its own turn as `Relay`, it would be useless to designate this other `DA` again as a `Relay` because it may duplicate-filter the message and therefore not relay it again, anyway. A `Relay` may, but does not have to, send a message multiple times in different timeslots if it has been designated to do so; such routing loops may result from limited meaningful choices based on the local part in an `RDCP Infrastructure` topology but do no contribute to the overall coverage and may be suboptimal under duty-cycle considerations.

To avoid LoRa packet collisions during regular relaying, each `Relay/Delay` `RDCP Header` field that actually designates a `Relay` (i.e., does not use the magic value of 0xE) also assigns a `Delay` in number of `Timeslots` according to the following table:

| `Timeslot` | Future `Timeslots` | Sending `DA`   | Designated `Relay` | Assigned `Delay` |
| :--------: | :----------------: | :------------: | :----------------: | :--------------: |
| 0          | 8                  | `Entry Point`  | `First Hop` 1      | 0                |
|            |                    |                | `First Hop` 2      | 1                |
|            |                    |                | `First Hop` 3      | 2                |
| 1          | 7                  | `First Hop` 1  | `Second Hop` 1     | 2                |
|            |                    |                | `Second Hop` 2     | 3                |
| 2          | 6                  | `First Hop` 2  | `Second Hop` 3     | 3                |
|            |                    |                | `Second Hop` 4     | 4                |
| 3          | 5                  | `First Hop` 3  | 0xE                | 4                |
| 4          | 4                  | `Second Hop` 1 | `Third Hop` 1      | 3                |
| 5          | 3                  | `Second Hop` 2 | 0xE                | 2                |
| 6          | 2                  | `Second Hop` 3 | 0xE                | 1                |
| 7          | 1                  | `Second Hop` 4 | 0xE                | 0                |
| 8          | 0                  | `Third Hop` 1  | 0xE                | 0xE              |

When a `DA` receives an `RDCP Message` in which it is designated as `Relay`, it knows that it is a `First Hop` if the first designated `Relay` is assigned a `Delay` of 0 and three `Relays` have been designated. Similarly, it knows that it is a `Second Hop` if it was explicitly designated as one of two `Relays` with `Delays` greater than 0. A `DA` is a `Third Hop` if it is the only designated `Relay` with a `Delay` of 3. If `0xF` is used as a `Third Hop`, all receiving `DAs` that did not relay the same message before send in the same, final `Timeslot` despite this may cause LoRa packet collisions in parts of the overall topology. The rationale is that `Second Hops` are expected to be sufficient to completely cover the overall topology during regular operations; letting "everyone who has not done so far" relay at the end of the `propagation cycle` serves as a safety margin if previous `Relays` were chosen awkwardly or if parts of the overall topology are offline. Note that only those `DAs` will become such actual `Third Hops` that received the message from the first of the four `Second Hops`; otherwise, the message is ignored based on the `duplicate filtering` mechanism.

Similarly, when an `RDCP Message` is received, the unique combination of the two 4-bit values in the `Relay/Delay 1+2` fields of the `RDCP Header` indicates the current `Timeslot` in the `propagation cycle` of the current message. In combination with the initial value of the `Retransmission Counter` header field and the size (and therefore airtime) of the LoRa packet, each recipient can derive when the `propagation cycle` will end and the channel is expected to be free again. Note that a distinction between timeslots 2 and 4 should be based on the number of designated relays as both `Relay/Delay 1` header fields use a `Delay` of 3. Thus, if no suitable fourth `Second Hop` is available, the magic value of 0xD must be used for it.

New `RDCP Messages` must only be sent when the `propagation cycle` of a previous message has ended. Random delays and channel activity detection ("listen before talk") should additionally be applied before sending a new message, as other `DAs` might also have new messages queued.

Collisions may still occur if new messages are sent roughly at the same time in different parts of the topology (the classical "hidden node problem" where "listen before talk" inherently fails). Two kinds of "collisions" must be considered:

- A "real" LoRa packet collision means that neither message is received properly by an affected `DA`. For this reason, `end-user` generated messages of `RDCP Message Type` `CITIZEN REPORT` are sent with a new `Sequence Number` if they are not confirmed by an `HQ` device via an `ACKNOWLEDGMENT` within a reasonable timeframe. These 1st category collisions cannot be handled by `DAs` as they do not receive either of the involved messages properly.
- A `DA` receives a second `RDCP Message` before the `propagation cycle` of a first `RDCP Message` has finished. These 2nd category collisions can be handled by either of:
  - Postponing:
    - The `DA` handles the first received `RDCP Message` as it usually would (i.e., as designated `Relay` or simply as recipient).
    - Relaying the conflicting message uses the end of the first message's `propagation cycle` as scheduling baseline instead of the current point in time, i.e., the second `propagation cycle` is paused and resumed when the channel is free again.
  - Filtering:
    - If the receiving `DA` is a designated `Relay` for both messages, only one of them is relayed. `RDCP Message Types` are prioritized for this purpose, and `OFFICIAL ANNOUNCEMENTs` and `CRYPTOGRAPHIC SIGNATUREs` are chosen before `ACKNOWLEDGMENTs` and `CITIZEN REPORTs` (in this order). If no such prioritized message is affected, the one furthest in the `propagation cycle` is relayed.
    - If the receiving `DA` is a designated `Relay` for only one of the messages, it is *not* relayed. However, if the message to relay is an `OFFICIAL ANNOUNCEMENT` or `CRYPTOGRAPHIC SIGNATURE`, it is sent after the `propagation cycle` of the conflicting message has ended and a random delay is applied, just as if the affected `DA` was the `Entry Point` of the `OA`.
  - If the receiving `DA` is not a designated `Relay` for either message, the "collision" is ignored and both messages are processed as usual.
  - All `DAs` in an `RDCP Infrastructure` must implement the same strategy for handling 2nd category collisions.

The prioritized order of `RDCP Message Types` frist reflects that `OAs` and their accompanying `CRYPTOGRAPHIC SIGNATUREs` are not sent again if they are lost on the way, and the `HQ` device has no way to detect this situation. Second, missing `ACKNOWLEDGMENTs` also cause the message to be confirmed having to be re-sent, further increasing the load on a currently strained LoRa channel. Third, `end-user` generated `CITIZEN REPORT` messages are important, but implicitly re-sent in case of loss. Finally, a complete loss of other `RDCP Messages Types` does not impact the core functionality of the `RDCP Infrastructure`, so it is acceptable in high-load situations.

`MGs` must support using a preconfigured `DA` as the `Entry Point` for their `RDCP Messages`, usually the geographically nearest, which is set during the device provisioning process. If the primary `Entry Point` does not work (e.g., it does not send `ACKNOWLEDGMENTs` to `CITIZEN REPORTs`), the `MG` may fall back to preconfigured alternatives or use a `DA` from which it received `RDCP Messages` when the `DA` was the `Sender`. The latter case may also be the default for `MGs`, i.e., they can be roaming by using the `DA` as `Entry Point` that currently provides the best signal strength. Special-purpose `MGs` (i.e., those not for `end-users` and households, but for emergency services or maintenance personnel) may allow for a configuration of which `DA` to use as `Entry Point` at run-time.

## RDCP Message Authentication and Encryption

RDCP devices are provisioned by an `RDCP Infrastructure Operator`, which allows for storing cryptographic key material on each device.

For authenticating `RDCP Messages` whose `RDCP Payload` includes a cryptographic signature field of length *sig*, asymmetric cryptography is used and the `Public Key` of each device that is supposed to create such cryptographic signatures is stored on all other devices. This primarily affects `HQ` devices, but also includes `MGs` for maintenance operations by authorized personnel.

Current prototype implementations use Schnorr signatures with *sig* = 65 bytes because of their "small" size.

For `RDCP Messages` that integrate a cryptographic signature, it is created using a SHA-256 hash of the `static RDCP Header fields`, i.e., `Origin`, `Sequence Number`, `Destination`, `Message Type`, and `Payload Length`, as well as the `RDCP Payload` excluding the signature itself. Other `RDCP Header` fields are not used for the hash calculation as they will change during the `propagation cycle`.

For `CRYPTOGRAPHIC SIGNATURE` messages accompanying `OFFICIAL ANNOUNCEMENT` messages, the signature is created for a SHA-256 hash of the `RDCP Header` fields `Origin`, `Destination`, and `Message Type` as well as the `Subtype`, the `Reference Number`, the `Lifetime`, and the `More Fragments` field of the first fragment along with the concatenated `Contents` of all fragments (after their Unishox2-compression).

Separate `SIGNATURE` messages may be sent ahead of or after their corresponding `OAs`. It is recommended that `end-user` devices clearly indicate that an `OA's` authenticity could not be verified yet in the latter case, and remove `OAs` whose `SIGNATURE` was invalid. Implementations should also consider to remove `OAs` for which an accompanying `SIGNATURE` does not arrive within reasonable time.

For `CITIZEN REPORTs`, a shared secret 256-bit key specific for each non-`HQ` device (which is also stored on all `HQ` devices) is used for AES-256-GCM authenticated encryption using the `static RDCP Header` fields (see above) as _additional data_ and the original `RDCP Payload` as plaintext, and then replacing the `RDCP Payload` with the resulting ciphertext. Note that the overall `RDCP Payload` size maximum must not be exceeded, also accounting for the 16-byte AES-256-GCM authentication tag. A unique IV is derived from the `static RDCP Header` fields per message.

For `OFFICIAL ANNOUNCEMENTs` that are sent to a unicast address, the same authenticated encryption procedure is used as for `CITIZEN REPORTs`. `End-user` devices must verify the authenticity of received `OAs` sent to their unicast `Destination` address first, and only then can process decrypted subheader fields such as `Subtype`.

## Duty Cycle and Legal Considerations

- RDCP is designed to be used over LoRa channels, although other underlying LPWAN technologies could probably be used.
- The general assumption therefore is that radio bands are used for which no dedicated license or frequency allocation is required.
- However, such ISM / SRD bands typically have duty cycle regulations that must not be violated.
- In its `non-crisis mode`, RDCP (or, more precisely, its prototype implementations) is a very low-traffic protocol as only public event announcements and telemetry/management data are transmitted.
- In its `crisis mode`, where also `DAs` and `MGs` can be used to bring new messages into the mesh, traffic can increase arbitrarily in theory.
- We assume that in practice the traffic volume will still be rather low simply because we do not expect masses of `OFFICIAL ANNOUNCEMENTs` by public authorities or `CITIZEN REPORTs` by `end-users`, based on workshops held with autorities and citizens who experienced a real-world crisis in September, 2023.
- However, one part of the prototype deployment in the ROLORAN research project in the `Neuhaus Scenario` is to gather telemetry data that gives more detailed insight into real-world usage during disaster exercises and, eventually, real crises.
- Given the involvement of public authorities and emergency services, we will use these project results to determine whether dedicated frequencies or other special permits seem to be necessary or useful, and then update our recommendations for other RDCP `Scenarios`.
