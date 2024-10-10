
## Functional Requirements

-   **Buy and Sell Stocks**: The system should allow buying and selling stocks based on the  **spread**  (highest buying price crosses the lowest selling price).
-   **Cancel Orders**: Clients should be able to cancel previously placed orders.
-   **Market Data Inspection**: Users should be able to quickly see the  **volume at a given limit price**  (buy or sell orders).

## Capacity Estimates

-   **Message Volume**: The stock market processes  **100,000 messages per second**.
-   **Data Size**: With messages around 20 bytes each, the system handles about  **20 gigabytes per day**.

## Basic API Requirements

-   **Buy/Sell Security**: Users can buy or sell a security at a given price.
-   **Cancel Order**: Users can cancel an order using its  **order ID**.
-   **Get Prices/Volume**: Users can retrieve prices or volume at a given price.
-   **Broker Functionality**: If acting as a  **broker**  (like Robinhood), users should be able to view past orders, which would require a  **database**  (though the focus here is on a low-latency system).

## Algorithm and Design

-   **In-Memory Operations**: Since the system needs to execute orders in  **nanoseconds**, everything should happen  **in memory**  (no disk reads).
-   **Matching Engine**:
    -   **Core Component**: The  **matching engine**  is responsible for executing the  **limit order book**  algorithm.
    -   **Limit Order Book**: Tracks open buy and sell orders for a given stock and executes trades when the spread is crossed (buy price ≥ sell price).
    -   **Partial Orders**: Orders may be partially fulfilled if the volume doesn’t match exactly.

## Data Structures for Matching Engine

-   **Trees and Linked Lists**:
    -   **Two Trees**: One for  **buy orders**  and one for  **sell orders**.
    -   **Linked Lists**: Each tree node contains a linked list of orders at a specific price.
    -   **Hash Maps**:
        -   One hash map points to the start of each linked list at a given price.
        -   Another hash map helps with  **order cancellations**  by pointing to specific orders within the linked list.

## Broadcasting and Fairness

-   **UDP Multicast**: Used to broadcast order execution information to all clients and interested parties, ensuring  **fairness**  (all clients receive the information simultaneously).
-   **Why UDP?**:
    -   **Fairness**: Multicast ensures simultaneous delivery to multiple clients.
    -   **Speed**: UDP is faster than TCP because it doesn’t require handshakes.
    -   **Drawbacks**: UDP lacks sequence numbers and flow control, meaning messages can be lost.

## Retransmitters

-   **Role**:  **Retransmitter nodes**  keep a log of outgoing messages from the matching engine. If a client misses a message, it can request it from a retransmitter.
-   **New Clients**: Retransmitters can also help new clients catch up by sending them past messages.

## Fault Tolerance

-   **Secondary Matching Engine**: A  **backup matching engine**  is necessary to handle failures of the primary engine.
-   **Synchronization**:
    -   The  **primary matching engine**  determines the order of messages.
    -   The  **secondary matching engine**  listens to the primary’s broadcasts and updates its own state (limit order book) to stay in sync.
    -   **Failover**: If the primary fails, the secondary can take over with an up-to-date state.

## Sharding for Performance

-   **Sharding by Ticker**: The system could be  **sharded**  by ticker (e.g., Apple, Google, etc.) to improve performance.
-   **Trade-offs**: Sharding can reduce the ability to perform  **multi-ticker transactions**  (conditional trades across different stocks).

## Speed Optimization

-   **Latency is Key**: The faster the system, the more accurate the prices, and the fewer opportunities for arbitrage.
-   **Limiting Factor**: The system’s  **throughput**  is limited by its  **latency**. Faster processing allows more network messages to be handled, reducing congestion.

## Diagram Explanation

-   **Clients**: Clients send orders to  **order services**  (web servers).
-   **Order Services**: These services forward orders to the  **matching engine**.
-   **Matching Engine**:
    -   Processes orders using the  **limit book algorithm**.
    -   Sends messages to  **retransmitters**,  **backup matching engine**, and clients.
-   **Retransmitters**: Keep track of messages in case of message loss.
-   **Cancel Process**: A separate process/thread handles order cancellations.
-   **Microservices**: Other services (e.g., stream processing, fraud detection, databases) can be built on top of this core system.

## Final Remarks

-   **Focus on Latency**: The primary focus of the system is  **latency**  rather than distributed systems, although knowledge of network protocols, state machine replication, and UDP is essential.
-   **Additional Features**: Other layers, such as  **real-time notifications**  or  **database tracking**  of transactions, can be added later.
-   

---------
## Capacity Estimates

-   100,000 messages per second
-   Approximately 20 GB of data per day

## API Design

-   Buy security at a given price
-   Sell security at a given price
-   Cancel existing order
-   Get prices or volume at a given price
-   View past orders (if acting as a broker)

## Core Components

### Matching Engine

-   Encapsulates the functionality of the limit order book
-   Runs entirely in memory for low latency
-   Uses efficient data structures:
    -   Two trees (for buy and sell orders)
    -   Linked lists for orders at each price point
    -   Hash maps for quick access and cancellations

### Order Book Algorithm

-   Tracks open buy and sell orders
-   Executes trades when buy price meets or exceeds sell price
-   Partial order fulfillment possible

### Message Broadcasting

-   Uses UDP multicast for speed and fairness
-   Broadcasts executed orders and confirmations to all interested parties

### Retransmitter Nodes

-   Store logs of outgoing messages from the matching engine
-   Provide missed messages to clients upon request
-   Help new clients catch up with current state

### Backup Matching Engine

-   Runs alongside the primary matching engine
-   Updates its state based on broadcasted messages from the primary
-   Takes over if the primary fails

## System Design Considerations

1.  **In-Memory Processing**: Everything runs in memory to achieve nanosecond-level execution times.
    
2.  **Minimal Locking**: Avoid locking as much as possible for speed.
    
3.  **UDP Multicast**: Used for broadcasting messages quickly and fairly.
    
4.  **Fault Tolerance**: Implemented through a backup matching engine.
    
5.  **State Machine Replication**: Used to keep the backup in sync with the primary.
    
6.  **Sharding**: Possible to shard by stock tickers, but may limit multi-ticker transactions.
    
7.  **Latency Focus**: Improving latency directly improves throughput due to UDP’s lack of congestion control.
    

## Additional Components

-   Order services (web servers)
-   Cancel process (decoupled from matching engine)
-   Stream processing for fraud detection and analytics
-   MySQL database for financial transactions
-   Time series database for historical data

## Challenges and Considerations

-   Ensuring message ordering with UDP
-   Handling UDP packet loss
-   Balancing speed and reliability
-   Managing multi-ticker transactions in a sharded system