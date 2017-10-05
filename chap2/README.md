# Application Layer

## Principles of Network Applications

### Network Application Architectures

:bookmark: `client-server`: this architecture is the most popular one in use. A server is a host that is always on. This means that the downtime of the host must be as minimal as possible. A client is also a host that requests something from a server. A server, however, is required to maintain a static IP address so that clients can detect it.

:bookmark: `peer-to-peer`: this architecture is mostly used in file sharing systems where more users can contribute to more bandwidth. All hosts are dynamic hosts and they are similiar to each other. It is also self-scalable.

### Processes Communication

- Goal: send a message from an application of a host to another application of another host. Endpoints are applications.
- The OS on which the application runs on, exposes API to access the socket. A `socket` is an abstraction to all OSes which indicate the door from the application to transport layer (review layers from chapter 1).

![process-communication](https://user-images.githubusercontent.com/22194121/30956958-02f07c18-a463-11e7-831d-e32d1ef75f36.png)

- Client-Server communication: This kind of communication involves 2 parties only. The one that initiates the communication is the client. The other is the server.

- All those clients and servers use sockets as their means for communication. In addition, for a process to send packets to another process, there must be an identification signature for those 2. The most common identification signatures are the couple of IP address of the host and the network port that the other process is holding (A port is limited to 16 bits only with the first 1024 ports reserved exclusively for the OS).

### Transport Services Available to Applications

1. **Reliable Data Transfer**

- This mechanism (protocol) is implemented at the Transport Layer. It is to ensure that packets are sent to the final destination (if there is a packet loss, it will detect it and re-send until the other can respond that it has received the message). If it cannot make sure the other end can receive such a message, timeout will happen.
- For `loss-tolerant` applications, this service is not really worth it because it might slow down the application and the integrity of the message is not strictly required.
- However, there are some special cases where the developers/programmers want to use their own mechanism/protocol of reliability, they will ignore this service at the transport layer and implement their own method at the application level.

2. **Throughput**

- This service guarantees a minimum throughput for some special applications that are categorized as `bandwidth-sensitive` ones (video call, phone call, ...).
- The `elastic` applications are not fond of this service because they do not require a reserved bandwidth.
- This is indirectly implemented through congestion control.

3. **Timing**

- This service guarantees that a packet is sent to the destination with maximum `t` time.
- This is indirectly implemented through congestion control.

4. **Security**

- Encryption is also available for packets being sent out.

### Transport Services Provided by the Internet

|Features|TCP|UDP|
|---|:---:|:---:|
|Encryption|:heavy_multiplication_x:|:heavy_multiplication_x:|
|Reliable Data Transfer|:heavy_check_mark:|:heavy_multiplication_x:|
|Congestion Control|:heavy_check_mark:|:heavy_multiplication_x:|
|Duplex|:heavy_check_mark:|:heavy_check_mark:|
|Port|

- `Transport Control Protocol` or `TCP`:
    - Connection-oriented: There are communications before the real message is being sent. Those communications are called `handshakes`. After that, it is said that a `TCP connection` is established and the two parties can start talking real shits.
    - Reliable Data Transfer: Guarantee no missing or duplicate packets.
    - `SSL`: This is an enhancement to TCP. It acts like a middle layer between the application and the transport. Besides, it also provides data integrity and end-point authentication.

- `User Datagram Protocol` or `UDP`:
    - It does not provide nearly any services.
    - Its manifesto is to keep the packet as light as possible. The intention is to defer the responsibility of handling services to application layer so user is more flexible in choosing their own implementation.

## The Web and HTTP

### Overview of HTTP
- A `Web page` consists of objects. An `object` is a file such as HTML file, image JPEG, video clip.
- The client will send a HTTP `request` to the server and the server will receive it and respond with a HTTP `response` (usually this reponse contains the web page).
- HTTP uses TCP instead of UDP.
- HTTP is a `stateless` protocol because it does not save its state when communicating. The server and client does not assume any thing before a communication.
- The normal port for HTTP is port `80`.

### Non-persistent and Persistent Connections

1. HTTP with Non-persistent Connections:

- Each object required by the client initiates individual handshake. This means if there are 10 objects, there will be 10 handshakes.
- Because of that, the server always closes the TCP connection after each response.
- Let `RTT` or `Round Trip Time` be the duration between the moment the request is sent and the instance the client receives a response. Let the `FTT` or `File Trip Time` be the duration to transfer a file from the server to the client and it is the same for every file. To send 1 file: we need `2 * RTT + FTT`. For 10 files: we need `20 * RTT + 10 * FTT`.

![non-persistent-http](https://user-images.githubusercontent.com/22194121/31216199-f9a58232-a9dc-11e7-94af-4bd9a7f3acbb.png)

2. HTTP with Persistent Connections:
- For persistent connections, the server will leave the TCP connection open (after a period of idle time, it will close). The client does not need to handshake with the server again, it just sends the request and the server will respond.
- For 1 file: There are still `2 * RTT + FTT`. For 10 files: `11 * RTT + 10 * FTT` (the first `RTT` is a hello handshake, the second `RTT` is to request the first object. We still need 9 more so adding 9 more quests: 2 + 9 = 11)

### HTTP Message Format

1. HTTP Request Message:

```http
REQUEST-TYPE /do/something HTTP/1.1
Host: https://www.api.call 
Header-1: Value-1
Header-2: Value-2
...
Header-3: Value-3

Content is here. Write something
```

- `REQUEST-TYPE`: There are many types of requests.
    - `GET`: Get a resource from the server.
    - `POST`: Create a new resource in the server.
    - `DELETE`: Delete a resource in the server.
    - `PUT`: Change/Edit a resource in the server.
    - `HEAD`: This is equivalent to `GET` except that it does not need any content. This is to check a service, checking edit history, etc.
    - `OPTIONS`: Expose what kind of options for CORS. (Will be updated later)
- `/do/something`: Path to the resource on the server.
- `HTTP/1.1`: Version of the HTTP.
- `Host`: A special type of header that contains the value of the server host.
- `Header-x: Value-x`: Value of header.
- The final thing is the body. It contains the content.

<table>
    <tr>
        <th>0</th>
        <th>1</th> 
        <th>2</th>
        <th>3</th> 
        <th>4</th>
        <th>5</th> 
        <th>6</th>
        <th>7</th> 
        <th>8</th>
        <th>9</th> 
        <th>10</th>
        <th>11</th> 
        <th>12</th>
        <th>13</th> 
        <th>14</th>
        <th>15</th> 
        <th>16</th>
        <th>17</th> 
        <th>18</th>
        <th>19</th> 
        <th>20</th>
        <th>21</th> 
        <th>22</th>
        <th>23</th> 
        <th>24</th>
        <th>25</th> 
        <th>26</th>
        <th>27</th> 
        <th>28</th>
        <th>29</th> 
        <th>30</th>
        <th>31</th> 
    </tr>
</table>
