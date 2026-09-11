# Async Sockets C++

A lightweight, thread-based asynchronous TCP and UDP socket framework written in modern C++.

The project provides simple client-server abstractions over low-level socket programming, allowing applications to establish TCP connections, exchange reliable stream-based messages, and communicate through connectionless UDP datagrams without directly managing the complete socket lifecycle.

The project is designed for Linux/Unix-like environments and focuses on practical networking concepts such as socket programming, asynchronous communication, multithreading, callbacks, resource management, and client-server architecture.

---

## Features

* TCP client and server communication
* UDP client and server communication
* Thread-based asynchronous socket handling
* Callback-based event handling
* Multiple concurrent TCP connections
* Connection and disconnection callbacks
* TCP message sending and receiving
* UDP datagram communication
* Error callbacks for network failures
* Linux/POSIX socket-based implementation
* Lightweight socket abstraction
* Simple API for creating network applications
* Support for concurrent network operations
* RAII-oriented resource management
* C++11/14/17 compatible design

---

## Architecture

```text
                    Application
                         |
                +--------+--------+
                |                 |
            TCP Layer         UDP Layer
                |                 |
        +-------+-------+   +-----+------+
        |               |   |            |
    TCP Client      TCP Server  UDP Client  UDP Server
        |               |   |            |
        +---------------+---+------------+
                        |
                 Socket Abstraction
                        |
                 POSIX Socket API
                        |
              Linux / Unix Networking
```

The framework separates application-level callbacks from low-level socket operations.

TCP and UDP provide different communication models:

### TCP

TCP provides a reliable, connection-oriented byte stream.

```text
Client
   |
   | connect()
   v
Server
   |
   | send()/recv()
   |
Reliable communication
```

### UDP

UDP provides connectionless datagram communication with lower protocol overhead.

```text
Client
   |
   | sendto()
   v
Server
   |
   | recvfrom()
   |
Datagram communication
```

---

## Why TCP and UDP?

The project demonstrates the practical difference between the two major transport protocols.

| TCP                                 | UDP                                          |
| ----------------------------------- | -------------------------------------------- |
| Connection-oriented                 | Connectionless                               |
| Reliable delivery                   | No delivery guarantee                        |
| Ordered byte stream                 | Individual datagrams                         |
| Retransmission handled by protocol  | No automatic retransmission                  |
| Higher protocol overhead            | Lower overhead                               |
| Suitable for reliable communication | Suitable for latency-sensitive communication |

---

## Threading Model

The framework uses a thread-based asynchronous model.

For TCP, the server can accept multiple client connections and handle each connection independently.

```text
                    TCP Server
                        |
                 accept connection
                        |
        +---------------+---------------+
        |               |               |
     Thread 1        Thread 2        Thread 3
        |               |               |
     Client 1        Client 2        Client 3
```

This prevents one client connection from blocking the processing of other connections.

Because shared state can be accessed concurrently, applications using shared resources should synchronize access using mechanisms such as:

```cpp
std::mutex
std::lock_guard
std::unique_lock
std::atomic
```

---

## Callback-Based API

The framework uses callbacks to allow applications to react to network events.

Example:

```cpp
TCPSocket tcpSocket;

tcpSocket.Connect("127.0.0.1", 8888, [&]() {
    std::cout << "Connected to server" << std::endl;

    tcpSocket.Send("Hello Server!");
});
```

Callbacks can be used to handle events such as:

* Connection established
* Message received
* Connection closed
* Network error

This allows application logic to remain separate from socket-management logic.

---

# Project Structure

```text
async-sockets-cpp/
│
├── async-sockets/
│   ├── tcpsocket.hpp
│   ├── udpsocket.hpp
│   └── ...
│
├── examples/
│   ├── tcp-client.cpp
│   ├── tcp-server.cpp
│   ├── udp-client.cpp
│   └── udp-server.cpp
│
├── img/
│
├── LICENSE
├── README.md
└── .gitignore
```

---

# TCP Example

## TCP Server

```cpp
#include "tcpsocket.hpp"
#include <iostream>

int main()
{
    TCPServer server;

    server.onClientConnected = [](int client) {
        std::cout << "Client connected" << std::endl;
    };

    server.onMessageReceived = [](int client, std::string message) {
        std::cout << "Received: " << message << std::endl;
    };

    server.Bind(8888);

    return 0;
}
```

## TCP Client

```cpp
#include "tcpsocket.hpp"
#include <iostream>

int main()
{
    TCPSocket client;

    client.Connect("127.0.0.1", 8888, [&]() {
        std::cout << "Connected to server" << std::endl;

        client.Send("Hello Server!");
    });

    return 0;
}
```

---

# UDP Example

## UDP Server

```cpp
#include "udpsocket.hpp"
#include <iostream>

int main()
{
    UDPServer server;

    server.onMessageReceived =
        [&](std::string message, std::string address, uint16_t port)
    {
        std::cout << address << ":" << port
                  << " -> " << message << std::endl;

        server.SendTo(
            "Message received",
            address,
            port
        );
    };

    server.Bind(8888);

    return 0;
}
```

## UDP Client

```cpp
#include "udpsocket.hpp"
#include <iostream>

int main()
{
    UDPClient client;

    client.SendTo(
        "Hello UDP Server!",
        "127.0.0.1",
        8888
    );

    return 0;
}
```

---

# Building the Examples

## Requirements

* Linux
* GCC or Clang
* C++11 or newer
* POSIX socket support
* Make

Check your compiler:

```bash
g++ --version
```

Clone the repository:

```bash
git clone https://github.com/Rudresh757/async-sockets-cpp.git
cd async-sockets-cpp
```

Build the examples:

```bash
cd examples
make
```

---

# Running the TCP Example

Start the server:

```bash
./tcp-server
```

Open another terminal and start the client:

```bash
./tcp-client
```

The client connects to the TCP server and exchanges messages through the socket abstraction.

---

# Running the UDP Example

Start the UDP server:

```bash
./udp-server
```

In another terminal:

```bash
./udp-client
```

The client sends a UDP datagram to the server using the configured IP address and port.

---

# Networking Concepts Demonstrated

This project provides hands-on implementation of:

### Socket Lifecycle

```text
socket()
   ↓
bind()
   ↓
listen()
   ↓
accept()
   ↓
send()/recv()
   ↓
close()
```

For clients:

```text
socket()
   ↓
connect()
   ↓
send()/recv()
   ↓
close()
```

For UDP:

```text
socket()
   ↓
bind()
   ↓
sendto()/recvfrom()
   ↓
close()
```

---

# Concurrency

The framework uses C++ threads to handle concurrent network operations.

Important concurrency concepts demonstrated include:

* Thread creation
* Thread lifecycle
* Shared state
* Mutex-based synchronization
* Atomic operations
* Race-condition prevention
* Concurrent client handling
* Graceful connection termination

---

# Error Handling

Network operations can fail for many reasons, including:

* Connection refused
* Connection reset
* Invalid address
* Port unavailable
* Client disconnection
* Send/receive failure
* Socket creation failure

The framework exposes error callbacks so that applications can handle network failures without tightly coupling application logic to low-level socket operations.

---

# Performance Considerations

The framework follows a lightweight thread-based approach rather than requiring a large networking runtime.

The main design goals are:

* Low abstraction overhead
* Simple API
* Concurrent connections
* Asynchronous callbacks
* Lightweight TCP/UDP communication

Performance should always be evaluated under the workload and hardware being used rather than relying on fixed benchmark numbers.

---

# Debugging Network Traffic

Network traffic can be inspected using Linux networking tools.

### Check listening sockets

```bash
ss -lntup
```

### Capture TCP traffic

```bash
sudo tcpdump -i lo tcp port 8888
```

### Capture UDP traffic

```bash
sudo tcpdump -i lo udp port 8888
```

### Wireshark

Wireshark can be used to inspect:

* TCP connection establishment
* TCP packets
* UDP datagrams
* Source/destination addresses
* Ports
* Packet timing
* Network errors

---

# Future Improvements

The current thread-based architecture provides a simple foundation for asynchronous networking.

Potential improvements include:

* [ ] CMake build system
* [ ] GoogleTest test suite
* [ ] GitHub Actions CI
* [ ] RAII-based socket wrapper
* [ ] Improved buffer management
* [ ] TCP message framing
* [ ] Connection timeout handling
* [ ] Graceful shutdown using POSIX signals
* [ ] Thread pool
* [ ] `epoll`-based event loop
* [ ] Non-blocking socket mode
* [ ] Reactor pattern
* [ ] Connection statistics
* [ ] Network performance benchmarks
* [ ] Address sanitization and bounds checking
* [ ] TLS/SSL support

---

# Technical Concepts

This project provides practical experience with:

```text
C++17
Object-Oriented Programming
STL
RAII
Resource Management
Multithreading
Mutexes
Atomic Operations
Callbacks
TCP/IP
UDP
Socket Programming
Client-Server Architecture
POSIX APIs
Linux Networking
Concurrent Programming
Network Error Handling
```

---

# Security Notice

This project is intended for learning and experimentation with C++ network programming.

Do not expose a development server directly to the public internet without implementing appropriate security controls such as input validation, authentication, encryption, connection limits, and safe buffer handling.

---

# License

This project is released under the MIT License.

See [LICENSE](LICENSE) for details.

---

# Author

**Rudresh**

GitHub:
https://github.com/Rudresh757

---

## Project Goals

The main goal of this project is to understand how asynchronous TCP and UDP communication can be implemented in C++ using low-level socket APIs while applying modern C++ concepts such as object-oriented design, multithreading, resource management, and callback-based event handling.

The project can serve as a foundation for higher-level systems such as:

* Chat applications
* Multiplayer networking
* Telemetry systems
* Industrial communication
* Distributed services
* Real-time data systems
* Network monitoring tools
