# TX Scheduling on RDCP Devices

Keeping LoRa channel usage low is of paramount importance for RDCP implementations, both to get important messages through quickly and to comply with duty cycle regulations.

This documents summarizes selected aspects and implementation concept details for TX scheduling on RDCP devices.

## Types of RDCP Devices

DAs use two LoRa channels (roughly: 433 MHz and 868 MHz), MGs only one LoRa channel (868 MHz).

Thus, DAs adequately implement the concept discussed below twice, i.e., independent for each LoRa channel -- although receiving on one LoRa channel may trigger a message to be scheduled for sending on the other LoRa channel.

HQ devices use MGs as LoRa modems; thus, like MGs, they are limited to a single 868 MHz channel in the vicinity of at least one DA.

## Channel Free Estimator (CFEst)

Each RDCP device keeps track of its own time; this is usually based on a monotonic clock with at least milliseconds resolution since device start. Wallclock time should only be used on devices that support it natively (e.g., HQ devices).

Each device derives a Channel Free Estimator (`CFEst`) timestamp and updates it when one of the events marked with (\*) below occur as outlined. `CFEst` represents the point in time (as an absolute timestamp from the device's point of view) when the used LoRa channel is expected to be free (again) and thus can be used for sending new waiting/scheduled messages; usually, but not always, additionally Channel Activity Detection (CAD) is applied to make as sure as possible that the LoRa channel is really free before starting to transmit.

Random delays in the magnitude of seconds are used in combination with CAD to avoid that multiple devices in the same collision domain start sending at the same time; such collisions can still occur, as their likelihood can only be reduced stochastically, and must be handled as outlined in the [RDCP Specs](https://github.com/roloran/RDCP-Specs).

`CFEst` is set as follows:

- `CFEst` is initially set to 0 or the current time when the device starts operating. RDCP devices are expected to not start sending anything immediately after turning on, but instead apply a minimum delay of 30 seconds. During this grace period, RDCP Messages may be received that indicate ongoing `propagation cycles` (see [RDCP Specs](https://github.com/roloran/RDCP-Specs)) to be honored.

- On receiving a LoRa packet (RX),
-- `CFEst` is either updated to the end of the `propagation cycle` of the received RDCP Message + a random delay between 1 and 5 seconds (\*). Receiving an RDCP message also resets the CAD retry counter (see below).
-- or `CFEst` is left untouched if the received LoRa Packet was not an RDCP Message (i.e., the channel was used by other LoRa devices, but not RDCP devices, so there is no propagation cycle to consider).
-- NB: If an RDCP Message was received while the LoRa channel is still supposedly busy with another RDCP Message, collision handling according to [RDCP Specs](https://github.com/roloran/RDCP-Specs) should be triggered. (\*)

- On sending an RDCP message (TX),
-- `CFEst` is updated to the end of the `propagation cycle` if the device successfully started transmitting the message (e.g., after CAD). (\*)
-- If CAD is used and signals that the LoRa channel is not free on first try, up to 4 immediate CAD retries are performed.
-- On the 5th CAD retry, `CFEst` is increased by 20 + random(1--5) seconds. (\*)
-- CAD retries 6--9 happen without changes to `CFEst`, similar to tries 1--4, after toggling the LoRa radio back to RX mode for the 21--25 seconds delay (as is done on the 5th retry).
-- CAD retries 10--14 are similar to the 5th retry, with a 30 + random(1--5) seconds delay on each retry. (\*)
-- On the 15th retry, the message is forcefully sent (ignoring an occupied channel), and `CFEst` is set to the end of its propagation cycle. (\*)

(\*) means that all other messages in the TX queue need to be re-scheduled based on the new `CFEst` value.

CAD with its current implementation apparently takes about 250 ms of time, so 15 CAD attempts delay by about 3.75 seconds. With the additional delays of 21--25 seconds + 5 * (31--35) seconds, a single message can be delayed by 179.75 -- 203.75 seconds purely based on CAD. RDCP Messages that come in-between (RX) can delay even longer (depending on their propagation cycle and how many of them are received). This will be used below in the context of dropping (throwing away) non-important outgoing messages.

For sending certain RDCP Messages (TX), CAD is not used on purpose. This currently only applies to DAs designated as `Relays` for individual RDCP Messages; in this case, the `Relay` must stick to its assigned relaying timeslot to avoid collisions with other `Relays`.

As receiving an RDCP Message resets the CAD retry counter, forced sending an RDCP Message on the 15th retry can only happen if the used LoRa channel is excessively busy with other LoRa, non-RDCP devices; using a different channel should be considered as a better solution if this situation occurs more than sporadically. 

## TX Queue

Outgoing RDCP Messages (where the current device is the "Sender" in the RDCP Header) that should be sent either in an upcoming `timeslot` or as soon as possible (i.e., when the LoRa channel is supposed to be free) are stored in the `TX Queue`. There are further queues and scheduling functions for RDCP Messages that should be transmitted at a later point in time, so `TX Queue` is the nomenclature for the list of RDCP Messages that should be sent very soon, i.e., when the channel is considered free or their scheduled point in time has been reached (e.g., when relaying, the channel is not considered free for new messages, but a `Relay` needs to transmit anyway when its `timeslot` starts). Theoretically, at most a single RDCP Message should be in this queue, and while multiple queue slots can be supported technically, their use indicates an undesired congestion of the overall RDCP topology or a too-high rate of new RDCP Message creation in the current device given the current overall RDCP Infrastructure state.

The `TX Queue` provides the following functionality:

- New messages can be initially *add*ed to the `TX Queue`, and their scheduled point in time for transmission is max(`now`, `CFEst`) unless explicitly given otherwise (e.g., start of a `Relay` `timeslot`).

- *Re-scheduling* changes the scheduled point in time for transmission for all queue entries, usually into the future, triggered by the events marked with (\*) above. The re-scheduling time offset is the difference between the old and the new value of `CFEst`. Note that re-scheduling also drops (deletes, removes) outgoing messages that are not marked as important (see below) if their total delay due to re-scheduling is greater than 300 seconds or if they have been re-scheduled more than 20 times.

- A periodically called *loop* function checks whether it is time to transmit the next of the scheduled messages and triggers the actual TX functionality described below (either using CAD or transmitting immediately).

## TX Queue Scheduling Data

For each outgoing RDCP Message, the `TX Queue` stores data including

- the RDCP Message to be transmitted itself and its length.
- the timestamp of what point in time it was originally scheduled for transmission (not when it was added to the queue, but when it was supposed to be sent originally).
- the timestamp of what point in time it is currently scheduled for transmission (i.e., after re-scheduling).
- a counter of how often the message has been re-scheduled already.
- an indicator whether the RDCP Message is important and thus shall not be dropped even if it is delayed for a longer time or re-scheduled more often than outlined above; this currently applies to `OA` and `SIGNATURE` messages, but not, for example, to `CIRE` and `ACK` messages (as they will be sent again if lost somewhere on the way).
- an indicator whether the message must be sent on time independent of CAD status; this is important for `Relays` to stay within their `timeslot` in the `propagation cycle` of the RDCP Message.
- a callback selector of which `callback function` should be triggered once the message has been successfully sent. This is important for the chaining of several RDCP Messages, e.g. during responses to "fetch all new messages" requests or when older `OAs` are periodically retransmitted by DAs to MGs.
- the duration of a full `timeslot` for the message (including retransmissions and buffer time, but not a full `propagation cycle`).
- an indicator whether this message is currently being processed (singleton, i.e., only used for at most one message in the whole queue).

## TX Functionality

When an RDCP Message is up for transmission, the CAD-based sending approach outlined above is used unless the message is marked for immediate sending (i.e., independent of CAD status).

TX functionality includes auto-retransmission of messages, i.e., if the `Retransmission Counter` in the RDCP Header is larger than 0 and its transmission has finished, its `Retransmission Counter` is decremented, the `Checksum` field is updated accordingly, and the next retransmission is triggered under consideration of the 1.000 ms buffer time between retransmissions (i.e., depending on internal processing time, RDCP devices may pause for less than 1.000 ms between retransmitting a message, but the LoRa channel should be free for 1.000 ms between two retransmissions).

When the message has been transmitted completely, i.e., including all of its retransmissions, it is removed from the `TX Queue` and the selected `callback function` is called, so that, for example, the next RDCP Message in a chain of messages can be prepared.

No `callback function` is executed when a message is dropped from the `TX Queue` during re-scheduling.

## TX Scheduling ahead of time

Scheduling RDCP Messages for transmissions ahead of time (i.e., not for sending ASAP or in an upcoming `timeslot`) should be used rarely to avoid long queues of outgoing messages and therefore long-term occupation of the LoRa channel. However, it may be used by `Relays` to stick to their assigned `timeslot` in the `propagation cycle` and could also be used manually over Serial/UART when using an MG as a LoRa modem.

Schedules of this type are stored in a separate queue with its own periodic loop calls. It feeds outgoing messages into the `TX Queue` on time and removes them from its own queue. Usually, relative scheduling in number of milliseconds is used, and the indicator for the independence of CAD status as well as the importance status can be passed along.

This may, for example, be used to schedule complementing `SIGNATURE` RDCP Messages when its corresponding message has been brought on its way.

## Periodic Sending of Data

Currently, only DAs may periodically send data, such as `OA`/`SIGNATURE` retransmissions for MGs. This functionality typically uses the callback-based message chaining.

Messages to be sent periodically must be gathered independent of the other processes described here and added to either queue. However, such repeated messages must only be scheduled if the channel is currently free (see `CFEst`), once again to avoid building up longer queues of outgoing messages and to ensure that the LoRa channel can be shared fairly.

This concept basically also applies to HQ devices sending `TIMESTAMP` or `DA STATUS REQUEST` messages on a regular basis. However, HQ devices use independend queues for this purpose as they are implemented on separate hardware, utilizing a MG as LoRa modem and thus being decoupled from devices with integrated LoRa radios.

## Scheduling priorities

RDCP devices can implement additional priorities for the scheduling of messages to be sent. For example,

- DAs should prioritize sending `ACK` messages on their 868 MHz channel to avoid time-outs and re-sending attempts of MGs that previously sent a `CIRE` message.
- DAs in `crisis-mode` should expect an `ACK` from a HQ device soon after relaying a `CIRE` message received on their 868 MHz channel and thus not block their 868 MHz LoRa channel, e.g., with periodic `OA` retransmissions in such situations.
- DAs relaying a `CIRE` message on their 433 MHz LoRa channel can expect to relay an `ACK` message soon if they are in `crisis-mode`.
- DAs relaying an `OA` message can expect to be relaying a `SIGNATURE` message soon afterwards.

<!-- EOF -->