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

Both HTTP requests and responses use ASCII. To use Unicode, the client must encode the message to ASCII (sometimes not human-readable) and then the server will decode the message. The same thing is also applied in the case of files and objects.

1. HTTP Request Message:

```http
REQUEST-TYPE /do/something HTTP/1.1
Host: https://www.api.call 
Header-1: Value-1
Header-2: Value-2
...
Header-n: Value-n

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
- The final thing is the body. It contains the content.
- Another important thing is that there is an empty line between headers and the body.

2. HTTP Response Message:

```http
HTTP/1.1 200 MESSAGE
Header-1: Value-1
Header-2: Value-2
...
Header-n: Value-n

Body is here
```

- `HTTP/1.1`: Version of the HTTP response
- `200`: This is a typical code of success. There are more codes that indicate more things.
- `MESSAGE`: Descriptive message that goes along with the code.
- There is also a blank line between headers and the body.

:bookmark: There are some tools to play with HTTP: `telnet`, `curl`.

### User-Server Interaction: Cookies

|Step|:computer:<br>User|:repeat:<br>Flow|:satellite:<br>Server|
|---:|---:|:---:|:---|
|1|Want to get something|:arrow_right:|Receive & Create cookie: `3210`|
|2|Receive object|:arrow_left:<br>`Set-Cookie: 3210`|Send required object|
|3|Want another thing|:arrow_right:<br>`Cookie: 3210`|Receive & Check cookie
|4|Receive object|:arrow_left:|Verified & Send object|

- There are usually restrictions on what kind of resources a request can get/modify/edit. Hence, the server must be able to identify the host that initiates a request.
- However, authentication might be costly and there might be too many users to handle, the server must find a way to save a state of the user.
- Cookies are answers to that. In the 1st request, the server will send a `Set-Cookie` along with the headers so the host's browser will save it. After that, the host does not need to identify itself again (through password and username). It just needs to send the request with the cookie server has assigned to it.
- Cookies exposure can be critical to security and privacy. That's why the browser must keep it safe.

![cookies](https://user-images.githubusercontent.com/22194121/31222615-e7089582-a9f1-11e7-87bf-9afb45adbd26.png)

### Web Caching

![cache](https://user-images.githubusercontent.com/22194121/31222728-57756b60-a9f2-11e7-9233-f79cc4a086c7.png)

:bookmark: A `Web cache` or `proxy server` is a server that serves a HTTP request on behalf the original one.

:question: Why do we need this?
- This helps the Internet faster and more cost effective. Local ISPs and regional ISPs must pay to have access to the Tier-1 ISPs. It is really costly to send requests overseas and out of its network. Therefore, if there are an abundance of requests to the same server within a small duration of time (and the content does not change over that duration), it is better to save the response from the original server and serve the customer.

![cache-example](https://user-images.githubusercontent.com/22194121/31222788-8f09d4da-a9f2-11e7-8a43-cb29ab9a3a39.png)

- Example:
    - Request size: `1Mb`
    - LAN speed: `100Mbs`
    - Access link speed: `15Mbs`
    - Request rate: `15 requests/second`
    - Traffic intensity: `Bandwidth used / Max Bandwidth`
    - Round Trip Time: `2s`
    - Without cache:
        - Traffic Intensity on Access link: `Bandwidth Used / Access Link Speed = (Request Size * Request Rate) / 15Mbs = (1Mb/req * 15 req/s) / 15Mbs = 1`.
        - It uses up all the bandwidth and higher intensities will cause congestion.
    - With cache: `60%` go to origin servers and `40%` go to proxy.
        - Traffic Intensity on Access Link: `0.6 * 1 = 0.6`. (calculated from above)
        - This leaves some space if there are higher intensities.

- `Content Distribution Networks` or `CDNs` play important roles in web caching.

### Conditional `GET`

- Add `If-Modified-Since` to the header of the `GET` request.
- If the server checks that the resource is updated, it will send the standard message back to the client with success code `200`.
- Otherwise, it will send a response with an empty body and code `304` describing that `Not Modified` since the requested time.

## File Transfer Protocol: FTP

![ftp](https://user-images.githubusercontent.com/22194121/31225142-d9c0fbc2-a9fa-11e7-93e1-d3e557acd190.png)

- `FTP` is more or less the same as `HTTP`. The difference is that it uses 2 distinct connections: 1 for control information (port `20`) and 1 for data transferring (port `21`). This approach is called `out-of-band` and HTTP's approach to merge both control and data is called `in-band`.

![ftp-connections](https://user-images.githubusercontent.com/22194121/31225797-ed98611e-a9fd-11e7-8796-e8a13b1917b3.png)

- The client will initiate a TCP connection with with the server.
- After being accepted for communication, the client will identify itself with username password to access the server's file system. If the authentication is successful, then the client sends a request of types of transferring (`LIST`, `RETR`, `STOR`).
- Some commands used by the client to specify its requirements:
    - `USER`: username of the client
    - `PASS`: password of the client
    - `LIST`: List all files
    - `RETR filename`: Retrieve `filename` from the server.
    - `STOR filename`: Send the `filename` from local and store it on the server.

## Electronmic Mail in the Internet

- Sending email are more complex than accessing web pages.

- To send an email on the Internet, both of the receiver and the recipient must be online. There are too many variables contributing to the offline of both the sender and the receiver (blackout, unstable connection, low battery, ...). Therefore, we cannot send email directly from one host to another easily.
- Due to the unstability of the clients, mail servers are born to take the responsibility of being online all the time. Each of sender and receiver is attached to a mail server (or a distributed system of mail servers).

### SMTP: Simple Mail Transfer Protocol

- The process is as follows:
    1. A will compose her message on her local host.
    2. When A wants to send a message to B, she will contact her mail server and initiate a connection to send the email to that server (let's call A's server).
    3. Once the server has received the mail from A, it will store to its local file system.
    4. When the A's server has successfully received a mail from A, it will resolve the destination of the receiver's mail server (B's server). Then, it tries to contact the B's server to talk about a mail coming in hot. If the B's server is not available yet, A's server will try again after a coolout until the mail is sent successfully.
    5. B's server will store the mail and wait until B accesses the server.
    6. Finally, whenever the B access to his mailbox on mail's server, he will receive the new mails for him at his mail's server.

- SMPT uses port `25`.

![smtp](https://user-images.githubusercontent.com/22194121/31226958-553578f8-aa02-11e7-8f47-7a723d91aada.png)

- HTTP is called `pull protocol` because most of its requests are `GET` resources. SMTP is called `push protocol` because of its requests are sending mails to another (it does not require any resources from another machine).

### Mail Access Protocols

The SMTP is used in stage 2 and 4. Stage 6 is a little bit different.

![maps](https://user-images.githubusercontent.com/22194121/31227266-6c843ca0-aa03-11e7-8bb9-6bd8b1e6e8b9.png)

1. **POP3**

- Simple.
- When B initiates a TCP connection to get mails from B's server, the server will send mails to B and mark those mails as deleted.
- Cons: If B wants to access to his mails from multiple devices, he can't. The mails are stored on the first machine he used to access his server and it is the single point of failure (losing that device also results in loss of personal data).
- Additionally, POP3 does not save state of the mails. There are only 2 states: stored and deleted. Nothing more.

2. **IMAP**

- Complex.
- When B initiates a TCP connection to get mails from B's server, the server will send mails to B and mark those mails as read (state of the mail).
- IMAP also provides mechanism to categorize mails into folders and directories.
- Pros: User can access their mails from multiple devices because the mails are still on the server, not locally stored. (only deleted when users want to)

3. Web-base Emails

- Complex.
- You don't access the mails directly from an application but rather from a browser.
- The browser will connect to a web server. The web server will expose API call for mails manipulations. When the web server receives request of sending mails or getting mails, it sends SMTP (for sending) or IMAP (for receiving) mails to the real mail server. Just a man in the middle.

## DNS - The Internet's Directory Service

:bookmark: `DNS` or `Domain Name System` is a distributed system of servers which play the role of resolving host names to IPs. Host name such as `google.com` is for human-readable and memorable. IP addresses are for machines to initiate communication links.

![dns-hiearchy](https://user-images.githubusercontent.com/22194121/31232682-c8cfaaa6-aa14-11e7-871b-5d9ca9e05bc7.png)

:bookmark: A local or recursive DNS server will take responsibility from a DNS query to find out which IP address a host name belongs to.
- There are 3 types of DNS:
    1. Root DNS: This is the 1st place the local DNS look for.
        These servers do not contain IPs of the destinations but only contain IPs of middle servers. They only look for the last part of the host name such as `.com`, `.edu`, `.vn`. From this information, they will determine which servers might contain IPs of hosts ending with `.com` or `.edu`.
    2. Top-Level Domain DNS: (TLD)
        Each of these servers only serve 1 kind of the suffix (such as a dedicated server for `.com`, )or a reserved one for `.edu`. A server will not resolve names from other suffixes. It will look further to the name before the final suffix. For example, it will look into the part of `google` in `mail.google.com` and try to find where is the DNS of `google.com` is. The result might be both another TLD or the authoritative DNS.
    3. Authoritative DNS: This must be the final DNS server that can resolve a query (there might be many of these in query if there are many layers).
        These servers usually belong to organizations and institutions. Google will have their own DNS servers for this. A special thing about this server is that it does **NOT** recursively find further. For TLD and Root, those servers might recursively find the IPs but not for this one. If there is nothing on the server, it will send back an error immediately. Google DNS will resolve the part `mail` in `mail.google.com` to look for the real IP of the destination.

### DNS Caching

DNS Caching can happen almost on every DNS server (except for the Authoritative DNS because it does not request anything outside of its own system). This is even true for the hosts that send out DNS query (so they do not need to query the next time).

### DNS Records and Messages

(To be updated)

## Peer-To-Peer Applications

It's complex.



