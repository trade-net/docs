# Stock Exchange Simulation Design
This project aims to simulate the high level functionality of stock exchanges.
This project is a work in progress, and some of the designs discussed here have not yet been implemented.

## System Design
The proposed system:

![system_design](https://github.com/user-attachments/assets/e7560308-0eed-4ce4-840e-92276fcee922)

At the moment, the [matching-engine](https://github.com/trade-net/matching-engine/tree/main) is the only component completed.
It is a TCP server that accepts incoming order requests directly from clients,
which it either matches with existing orders in the limit order book, or inserts into the book (if the incoming order is a limit order that cannot yet be filled).

However, this is not ideal as clients should not be connecting to the matching engine directly. Instead, clients should connect to the exchange via its gateway.
The gateway then converts these requests into events and sends them to the sequencer to enforce determinism.
The sequencer should be UDP multicast based listening to both the gateway and the matching engine.
If the incoming event is accepted by the sequencer, it publishes to its downstream clients for the appropriate actions to be taken.

For simplicity, the sequencer should support 2 types of events: `OrderEvent` (from te gateway) and `TradeEvent` (from the matching engine).

When an `OrderEvent` is published by the sequencer, the matching engine should process the order and attempt to fill it.
If successful, it should send a proposed `TradeEvent` to the sequencer and store the event in some form of lookup/buffer.
When the `TradeEvent` is accepted and published by the sequencer, the matching engine can then complete the trade and remove the corresponding orders from the book.

The market data publisher and post trade notifer services should also consume from the sequencer.
This enables the market data publisher to publish all events externally, while the post trade notifier can be used to provide order updates to clients.
