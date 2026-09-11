# Asynchronous Sockets for C++
Simple, multithread-based(not thread safe), non-blocking asynchronous Client-Server classes in C++ for TCP & UDP. 
Creates a thread for every connection. Use `mutex`es or `atomic` variables to provide thread-safe functions.

```cpp
// Initialize a tcp socket.
TCPSocket tcpSocket;

// Connect to the host.
tcpSocket.Connect("127.0.0.1", 8888, [&] {
    cout << "Connected to the server successfully." << endl;

    // Send String:
    tcpSocket.Send("Hello Server!");
});
```
Super Easy!

**CPU & RAM Usages (with single tcp connection & with single udp server + client):**

Lightweight!

  
## Supported Platforms:
- Linux
- MacOS (not tested)
