# Transport Layer

## Introduction

:bookmark: `Logical Communication`: (Think of this as the opposite of physical communication) An abstraction of communication for application processes to talk with each other. From the previous chapter, application only talks via socket and they do not care about how distant in reality between 2 hosts. They only know that they can communicate through a socket and receive.

- Transport layer provides logical communication between application processes.
- Network layer provides logical communication between hosts.

## Multiplexing & Demultiplexing

:bookmark: `multiplexing`: Encapsulation based on 5 layers.
:bookmark: `demultiplexing`: Decapsulation off the 5 layers.

- Segment structure:

![segment](https://user-images.githubusercontent.com/22194121/31279046-adf325f6-aad1-11e7-8e5a-ec5069438436.png)

### Socket Programming with Multiplexing and Demultiplexing

- Connection-less:
    - Multiplexing:
        ```python
        import socket

        UDP_IP: str = '23.26.34.46'
        UDP_Port: int = 7917
        message: str = 'Hello from Connection-less communication'

        sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        sock.sendto(message, (UDP_IP, UDP_Port))
        ```
    - Demultiplexing:
        ```python
        import socket

        UDP_IP: str = '23.26.34.46'
        UDP_Port: int = 7917

        sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        sock.bind((UDP_IP, UDP_Port))

        while True:
            data, address = sock.recvfrom(1024)  # 1024 = buffer size
            print('Message: ' + str(data))
        ```

- Connection-Oriented: This type of communication focuses on the connection itself more than multiplexing or demultiplexing (both ends can do those 2 operations at the same time)
    - Client:
        ```python
        import socket

        TCP_IP: str = '23.26.34.46'
        TCP_Port: int = 7917
        message: str = 'Hello from Connection-Oriented communication'

        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.connect((TCP_IP, TCP_Port))  # Handshake
        sock.send(message)  # If accepted, send message
        data = sock.recv(1024)
        socket.close()

        print('Response: ' + str(data))
        ```
    - Server:
        ```python
        import socket

        TCP_IP: str = '23.26.34.46'
        TCP_Port: int = 7917
        message: str = 'Hi there from the server'

        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.bind((TCP_IP, TCP_Port))  # Specify which port IP and port for listening
        sock.listen(1)  # Wait for handshake initiation
        
        connection, address = sock.accept()  # Accept handshake
        while True:
            data = connection.recv(1024)
            if not data:
                break  # Time-out
            print('Request: ' + str(data))
            connection.send(message)

        connection.close()
        ```

## UDP: Connectionless Transport

- Talking with UDP is more or less the same as talking with network layer. There are only lightweight services without any "real" services implemented in this protocol.

- There are a few advantages: greater control over packets, no connection establishment (or use another self-implemented connection establishment), no connection state (or save the state at the developer's own wills), small packet header overhead (8 bytes UDP vs 20 bytes TCP)

- Some protocols and their choices of transport layer protocol to use:

![tcp-vs-udp](https://user-images.githubusercontent.com/22194121/31281196-f5a9f92c-aad8-11e7-9410-2b08f18f0c94.png)

### Segment Structure

![udp-structure](https://user-images.githubusercontent.com/22194121/31281384-778a0c48-aad9-11e7-988a-dab8e14481bd.png)

- `Length`: Length of the message.
- `Checksum`: A 16-bit one's complement number which indicates the 16-bit one's complement sum of pseudo headers (which contains equivalent information to the real IP header), UDP header (of course, we do not take into account the checksum header and must skip it, we haven't calculated it yet), data (padded with 0 to make sure it make up to octlet).
    - One's complement system: A binary arithmetic system such that a negative number is a bitwise negation of a positive number. (1 in 4-bit is `0001` and -1 in 4-bit is `1110`. Both `0000` and `1111` are `+0` and `-0` but still equal to `0`)
        - Addition: normal and the same as adding 2 binary numbers. If there is overflow, add that overflow (must be value `1`) to the sum (end-around carry).
        - Logic behind end-around carry: An `n-bit` one's complement can only represent `2^n - 1` values because `0` takes 2 representations (`00...0` and `11..1`). Hence, when there is an overflow (wrap around), its arithmetic calculation is missed by 1 unit. Adding `1` to it will make the calculation correct again.
    - After summing all 16-bit pieces, negate that number. The logic behind is that if the receiver wants to compare, it must proceed the same way the sender has and compare that to the checksum. However, thanks to the one's complement, let's say the sum is x. If we negate the sum we have the checksum, which is -x. The receiver only needs to sum all of that and compare to 0 (`-0` in this case, which is `11...1`).

## Principles of Reliable Data Transfers: RDT

### Building the Protocol

1. Reliable Data Transfer channel over a Perfectly Channel: `rdt1.0`

![rdt-1.0](https://user-images.githubusercontent.com/22194121/31305661-1e68d0be-ab69-11e7-961d-9e3d2a4cb402.png)

This is a simple case because the channel is realiable already. It does not have to implement anything extra.

2. Reliable Data Transfer over a Channel with Bit Errors: `rdt2.0`

- Error detection: use checksum.
- Receiver feedback: positive or 1 is `ACK` and negative or 0 is `NACK`.
- If there is an error or packet-loss, re-transmission is required.

![rdt-2.0](https://user-images.githubusercontent.com/22194121/31305731-f6f6dd6c-ab6a-11e7-96dd-6ce3c8d0918d.png)

- This scheme will add another state to the sender, which is called the waiting state. This state waits for the feedback packet from the receiver to determine whether the previous packet sent is accepted or not. If not, it will stay at that state and re-transmit the packet again until it is confirmed to be done.

- This protocol is also called `stop-and-wait`.

- This seems to work but not good enough. If somehow the feedback packet is corrupted, that means the receiver feedbacks with a NACK but sender receives ACK, then it will proceed to move on. Then, next packet it sends is something else and the receiver thought it was the packet which is previously corrupted. We need to protect the feedback packet too.

- Another problem is that when the receiver feedbacks with a NAK, the sender does not know at which state the receiver is receiving the packet (whether the previous or the current one is corrupted). Therefore, it is hard to track down what was happening.

**Improvement**: `rdt2.1`

- Instead of simple ACK and NACK, it wraps a `sequence number` to that. Hence, the receiver must receive sequentially that number.

- In this scheme, the feedback packet must also have a checksum field for corrupt detection. Besides, it must also send the sequence number with the packet to indicate the right state.

- This scheme removes the flaw of NAK in `2.0`. Instead of feedbacking with NAK, it sends ACK. If there are 2 ACKS in the same row with the same sequence number, there must be a problem at the receiver. The receiver somehow receives duplicate packet.

- Sender:
![rdt-2.1-sender](https://user-images.githubusercontent.com/22194121/31305877-f178c6f4-ab6d-11e7-97d0-5f08d53ec738.png)

- Receiver:
![rdt-2.1-receiver](https://user-images.githubusercontent.com/22194121/31305911-73f63a94-ab6e-11e7-98d1-1964a1f00c64.png)

**Improvement**: `rdt2.2`

### Go-Back-N Protocol

:bookmark: `Go-Back-N` protocol: The sender is allowed to transmit packets after packets without waiting for acknowledgements but with a limitation of `N` packets (before receiving some ACKs and move on).

![go-back-n](https://user-images.githubusercontent.com/22194121/31379253-f1e7119c-add7-11e7-9c53-ad719bab74df.png)

- Go-Back-N is also a sliding-window protocol because it slides a window of N packets every time the latest packet confirmed with an ACK.

- Flow control is 1 reason to limit to N-packet window.

- Sender:
![go-back-n-sender](https://user-images.githubusercontent.com/22194121/31379415-5ed3f40a-add8-11e7-9ff4-12381fd821ef.png)
    - Invocation from above: If the buffer is not full, it is allowed to send the data. Otherwise, the data packet is refused.
    - Receipt of an ACK: An ACK with its sequential number indicates cummulative acknowledgements. This means that every packet up until that sequence number has been received.
    - Time-out event: If there are no acks confirmed after an amount of time, sender will try to send all packets again. If there are some corrupted packets confirmed some time before this event, new packets were not sent because it will wait until this event to happen.

- Receiver:
    ![go-back-n-receiver](https://user-images.githubusercontent.com/22194121/31379769-7745193c-add9-11e7-90c7-c34c0ede1b78.png)
    - Receiver only keeps track of the last sequential number it has received (the `expected_seq_num`). This also means that if there are some packets with seq number larger than the expected one will be rejected.

- This protocol solves the problem of utilization from the normal `rdt` protocol.
- However, there are some problems with the bandwidth. If the window is pretty large, every timeout triggers a tremendous amount of packets to the receiver.

### Selective Repeat

![selective-repeat](https://user-images.githubusercontent.com/22194121/31380042-5d81da7a-adda-11e7-8608-c329c9307e45.png)

- Each packet will have its own timer. If the packet is sent but waiting ack is timed out, 

## TCP: Connection-Oriented Transport

### TCP Connection

- Full-duplex: Both parties can send and receive at the same time.
- Point to point: Single sender and receiver. There are no broadcasting or one to many relationships.

:bookmark: `Three-way Handshake`: Communication initiaton protocol of TCP.

:bookmark: `Maximum Segment Size` or `MSS`: the limit in size of a segment. (This does NOT include the header)
:bookmark: `Maximum Transmission Unit` or `MTU`: length of the largest link-layer frame. It is used to determine the MSS. (This DOES include the IP header or Ehthernet header from the link layer level).

- Components in a connection:
    - Buffer: contain data packets. (1 for sending and 1 for receiving).
    - Variables: IP, MSS, ...
    - Socket connection to the upper application layer.

### TCP Segment Structure

![tcp-segment-structure](https://user-images.githubusercontent.com/22194121/31383990-50e7324e-ade7-11e7-8c19-56d31abfb631.png)

- 32-bit `Sequence Number Field` and 32-bit `Acknowledgement Number Field`: reliable data transfer meta data.
    - `Sequence Number Field`: the order of the first byte of the data. If the whole data stream is 500 000 bytes and MSS is 1000 bytes, then it is segmented into 500 packets. The 1st packet is 0, 2nd packet is 1000, 3rd is 2000, ...
    - `Acknowledgement Number Field`: the expected sequence number to be received from the other end. In addition, the TCP will only acknowledge bytes up to the first missing byte, which is the byte indicated by this number (any packets after this segment beginning of this byte is excluded). This is called `cummulative acknowledgement`.
    - The sequence number field is based on the data from the perspective of the current host. The acknowledgement number field is based on the data from the other end. Therefore, sequence number field of host A is connected to acknowledgement number in host B.
    - Example:
        ![sqn-ack](https://user-images.githubusercontent.com/22194121/31385230-a05bde42-adec-11e7-9a20-35c6950db755.png)
- 16-bit `Receive Window`: Flow control variable
- 4-bit `Header Length`: Multiple of 32-bit header length. (If header is 128 bytes => Header Length = 4)
- `Options`: Optional field
- 6-bit Flags: contain 6 flags
    - `RST`, `SYN`, `FIN`: connection setup and teardown.
    - `PSH`: receiver should pass the data to the application layer if this bit is set.
    - `URG`: urgent data.

### Roundtrip-Time-Estimation and Timeout

- The TCP uses timeout and retransmission mechanism.
(to be updated)

### Reliable Data Transfer

- Single retransmission timer. (Multiple cause overhead)
- Sender actions:
    ![tcp-sender](https://user-images.githubusercontent.com/22194121/31385792-aba0ddaa-adee-11e7-9190-dc82af7e5bb5.png)
    - Data Received from application layer: Create a TCP segment with the next sequential number. If there is no timer, create a new one so that it can be retransmitted if there is anything wrong. The time-out is estimated from above. Then finally pass the segment to the network layer and increase the sequential number.
    - Timer timeout: Retransmit the oldest unacknowledged packet (Don't bother to retrasmit any other packet).
    - ACK received: If the ACK is larger than the `sendbase` (the first byte of the oldest unacknowledged packet), then update `sendbase` to that number. (We do not need to check any unacknowledged packet in between because TCP uses the cummulative acknowledgement so the ACK will always indicate all bytes before it are acked).
- Example:
    - Retramission due to a loss of packet:
        ![sample-retransmit-tcp](https://user-images.githubusercontent.com/22194121/31388021-daf05092-adf6-11e7-8be0-dadee1b8ffdd.png)
    - ACKs come late:
        ![sample-ack-late-tcp](https://user-images.githubusercontent.com/22194121/31388100-1cba58c4-adf7-11e7-9680-852514dd222d.png)
    - Cummulative acknowledgement avoids redundant retransmissions:
        ![sample-cum-ack-tcp](https://user-images.githubusercontent.com/22194121/31388185-714df904-adf7-11e7-93b6-525f595019c4.png)


### Flow Control

- In some scenarios, applications may be slow to read data from the buffer at the network layer. Hence, if TCP consumes too mcuh data, overflow will happen. Flow control comes to the rescue.
- Another common name for flow control is `speed-matching service`. This means that it matches the speed of connection to the speed to reading data from applications.
- Receiver:
    ![tcp-flow-receiver](https://user-images.githubusercontent.com/22194121/31388972-de1278ec-adf9-11e7-816f-5902dd78e79d.png)
    - `last_byte_received`: The index of the last byte received from the sender.
    - `last_byte_read`: The index of the last byte read by the application layer.
    - `data_buffer = last_byte_received - last_byte_read <= received_buffer`: The data part received but not read yet. (gray part)
    - `received_window = received_buffer - data_buffer`: The spare room where there is no data. Initially, `received_window = received_buffer`. (white part)
- Sender:
    - `last_byte_sent`: The index of the last byte sent to the receiver.
    - `last_byte_ack`: the index of the latest byte that was confirmed acked.
    - `sent_buffer = last_byte_sent - last_byte_ack`: This is the part where unacked packets belong.
- To make sure there is no overflow, the receiver feedbacks the sender with ack or packets indicating its `received_window` field (with the equivalent header). The sender's responsibility is to make sure that its `sent_buffer` must not exceed this `received_window`. Otherwise, there will be not enough space at the receiver's end.

- The only problem is that if the `received_window` is full, it will advertise the value to the sender. However, it will not update when the buffer is emptied out because the operation is only invoked when (A) it has data to send or (B) it has ACK to confirm. Hence, at the sender's end, it still sees that there is no space to send and it won't send any more. To counter this problem, sender will keep sending small packet to sender to check if there is space (after a timeout). Then, the sender will feedback with a new `received_window`.

### TCP Connection Management

- Three-way handshake protocol:
    ![three-way-handshake](https://user-images.githubusercontent.com/22194121/31391029-a35b079a-adff-11e7-9fe3-c658e819348d.png)
    1. Client sends a segment (without data) to the server indicating that it wants to talk:
        - SYN(chronized) bit: `1`.
        - Sequence Number field: a random `client_isn` number.
        - This is also called `SYN request`.
    2. Server receives the segment from (1) (assumed to be received). Server sends back a segment (without data) indicating that it agrees to establish the connection:
        - SYN bit: `1`.
        - ACK bit: `1`.
        - Acknowledgement Number field: a random `client_isn + 1` number.
        - Sequential Number Field: `server_isn`.
        - This is also called `SYNACK response`.
    3. Client receives the segment from (2). It now knows that the server has accepted its request. It sends the final segment in this scheme:
        - SYN bit: `0`.
        - ACK bit: `1`.
        - Acknowledgement number field: `server_isn + 1`.
        - This is also called `ACK response`.
    - After 3 steps above are done, both parties allocate buffers and variables to invest in the TCP connection.

- Closing TCP connection:
    ![tcp-close](https://user-images.githubusercontent.com/22194121/31391324-5e974d70-ae00-11e7-8a23-03d65c449a5a.png)
    1. Client sends a segment with field FIN(ish) is 1 to tell the server that it wants to ends thi connection.
    2. Server receives a segment from (1) and respond with and ACK.
    3. Then server will also send a segment with with FIN is 1.
    4. Client receives the segment and respond with ACK. The connection will be automatically closed after a timeout.
    - After step 4, all resources are deallocated.
    - This can be initiated from the server side.

### SYN Flooding

- During the stage 2 of the application, the server usually stores the connection in a SYN queue to wait for its turn for resource allocation.

- An attacker may spoof the IP and send tons of them to the server. Server will receive a lot of SYNs and try to put them in the queue to allocate their resources (MSS, time of initiating the connection). However, because those are faulty IPs, servers will never receive an ACK even when it sends out a SYNACK. Hence, those connections are left half open. Server may run out of resources or queue is overflown.

- `SYN cookies` are used to solve this problem. The server will use the IP and some information (secret number) to hash them to create the `server_isn` and put it into the sequential number. It will not place any of these connection in the queue yet. When it receives an ACK, it will try to match whether this ACK contains acknowledgement number of `server_isn + 1`. If it does, it will allocate now.

## Principles of Congestion Control

## TCP Congestion Control
