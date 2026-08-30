# Summary
---------
*Secure Python Server Project*

*This project is the beginning of building a secure server in Python. The goal is to create a server that can safely handle connections, protect sensitive information, and provide a foundation for adding authentication, encryption, logging, and other security features.*

*The project will start with a basic Python server and gradually introduce security measures such as encrypted HTTPS connections, secure authentication, input validation, error handling, access controls, and logging. Each feature will be added and tested individually to better understand how secure server software is designed and maintained.*

*The overall objective is to learn how the different components of a secure server work together while following good security practices throughout development.*

*Each component of the programme will be divided into different sections showcasing how each segment of the code works to show my understanding and allow you to understand my workings.*

*For now the building roadmap should look like this: Basic TCP server → HTTP server → HTTPS → authentication → sessions → permissions → rate limiting → logging → security tests.*

*The full programme will be linked at the end once finished, with my full permission to break it as you so desire.*

----------------------------
## Setting up the TCP server
----------------------------
*The first stage of this project focuses on building the foundation of a secure server by developing a custom TCP server using Python sockets. The server is responsible for creating and managing network connections, listening for incoming client requests, receiving transmitted data, and handling communication between connected systems. This stage focuses on understanding how servers operate at a lower level before introducing higher-level protocols and security mechanisms.*

*Key concepts explored in this stage include:*
- Creating TCP socket connections using Python
- Binding servers to specific IP addresses and ports
- Listening for and accepting client connections
- Receiving and processing incoming data
- Managing message boundaries using buffering
- Handling client disconnections safely

*A key challenge with raw TCP communication is that data is sent as a continuous stream rather than individual messages. The server therefore implements a basic buffer system to reconstruct complete messages and process them correctly.*

*This server will act as the base architecture for future development, where additional security features will be introduced, including:*
- HTTP request handling
- HTTPS encryption
- Authentication systems
- Session management
- Access control
- Rate limiting
- Security logging
- Vulnerability testing

```
import socket as soc

buffer = ""

server = soc.socket(soc.AF_INET, soc.SOCK_STREAM)
server.bind(("127.0.0.1", 8000))

server.listen(5)

while True:
    client, addr = server.accept()
    while True:
        client_msg = client.recv(1024)
        if not client_msg:
            break

        buffer += client_msg.decode()
        messages = buffer.split("\n")

        for message in messages[:-1]:
            print(message)
        buffer = messages[-1]

    client.send("Hello from e".encode())
    client.close()
```

*The client component is responsible for establishing communication with the server and transmitting data across a TCP connection. Using Python sockets, the client creates a connection to the server, accepts user input, formats the data into network-ready messages, and sends the information across the established connection.*

*This stage focuses on understanding the client-side processes involved in network communication, including:*
- Establishing TCP connections
- Communicating with remote services
- Encoding and transmitting data
- Handling message formatting
- Receiving server responses
- Closing connections correctly

*Because TCP operates as a continuous data stream, the client also ensures messages are correctly separated before being transmitted. This provides a foundation for later improvements involving structured communication protocols and secure data exchange.*

*Future development of the client will include implementing additional security features such as:*
- Encrypted communication
- Secure authentication
- Session handling
- Input validation
- Error handling
- Secure request management

*By developing both the server and client independently, this project provides a better understanding of how real-world networked applications communicate and where security weaknesses can occur.*

```
import socket

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(("127.0.0.1", 8000))

messages = input("Enter some crap this a test: ")

for message in messages:
    if not message.endswith("\n"):
        message += "\n"

    client.send(message.encode())

print(client.recv(1024).decode())

client.close()
```

## Setting up HTTP

---

*The second stage of this project focuses on developing a basic HTTP server on top of the TCP server created in the previous stage. Rather than relying on a high-level web framework, the server manually receives and interprets HTTP requests to better understand how HTTP operates over an underlying TCP connection.*

*The server has now been developed to recognise the structure of an HTTP request and separate the request into its main components. Incoming TCP data is stored in a byte buffer so that requests can be correctly reconstructed when the data arrives across multiple TCP packets.*

*The current server is responsible for:*

* Receiving HTTP requests over a TCP connection
* Maintaining a byte buffer for incoming network data
* Detecting the end of the HTTP header section using `\r\n\r\n`
* Separating the HTTP headers from the request body
* Decoding the header section for processing
* Separating the request line into the HTTP method, path, and version
* Extracting individual HTTP headers
* Identifying the `Content-Length` header when present
* Determining how much of the request body has already been received
* Continuing to receive data until the expected request body length has been reached
* Storing the completed request in a structured dictionary for easier processing

*A key part of this stage was understanding that TCP does not preserve HTTP message boundaries. A single HTTP request may therefore be received across multiple calls to `recv()`, while a single call may also contain both the HTTP headers and part or all of the request body. The server therefore uses a buffer and `Content-Length` to determine when the complete request has been received.*

*The completed request is stored using a structured format containing the request line, headers, and body. This provides a foundation for later stages where the server will need to inspect requests, validate incoming information, and make security decisions.*

### HTTP Server

*The HTTP server builds directly on the TCP server from the previous stage. The main difference is that incoming data is no longer treated as arbitrary messages. Instead, the server follows the structure defined by HTTP and interprets the different sections of the request.*

*The request line is separated into three components: the HTTP method, the requested path, and the HTTP version. The remaining header lines are then processed into a dictionary, allowing individual values such as `Host`, `Content-Type`, and `Content-Length` to be accessed directly.*

*When a request contains a body, the server uses the `Content-Length` header to determine how many bytes are expected. Any body data that has already arrived with the initial headers is retained, and additional calls to `recv()` are made when necessary until the expected amount of data has been received.*

*The resulting information is then stored in `request_data`, which provides a structured representation of the HTTP request and makes the information easier to work with in later stages of the project.*

```python
import socket as soc

server = soc.socket(soc.AF_INET, soc.SOCK_STREAM)
server.bind(("127.0.0.1", 8000))
server.listen(5)

while True:
    client, addr = server.accept()

    buffer = b""

    while b"\r\n\r\n" not in buffer:
        client_msg = client.recv(1024)

        if not client_msg:
            break

        print(f"Receiving data from {addr}:\n")
        buffer += client_msg

    header_section, body = buffer.split(b"\r\n\r\n", 1)

    messages = header_section.decode().split("\r\n")

    method, path, version = messages[0].split(" ")

    headers = {}

    for line in messages[1:]:
        key, value = line.split(":", 1)
        headers[key] = value.strip()

    content_len = int(headers.get("Content-Length", 0))

    while len(body) < content_len:
        body_data = client.recv(1024)

        if not body_data:
            break

        body += body_data

    request_data = {
        "method": method,
        "path": path,
        "version": version,
        "headers": headers,
        "body": body.decode()
    }

    print("Request Line:")
    for key, value in request_data.items():
        if key == "headers" or key == "body":
            continue
        print(f"{key}: {value}")

    print("\nHeaders:")
    for key, value in request_data["headers"].items():
        print(f"{key}: {value}")

    print(f"\nBody:\n{body.decode()}")

    client.send("Hello from e".encode())
    client.close()
```

### HTTP Client

*The client has been updated to generate and send properly structured HTTP requests to the server. Unlike the original TCP testing client, the current client sends a complete HTTP request containing a request line, headers, the required HTTP line endings, and, where necessary, a request body.*

*The client is mainly used as a testing tool for the HTTP server, allowing different request methods, paths, headers, and bodies to be sent and used to verify that the server correctly interprets the incoming data.*

```python
import socket

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(("127.0.0.1", 8000))

message = (
    "POST /login HTTP/1.1\r\n"
    "Host: localhost:8000\r\n"
    "Content-Type: application/x-www-form-urlencoded\r\n"
    "Content-Length: 21\r\n"
    "\r\n"
    "username=admin&pass=123"
)
client.send(message.encode())

print(client.recv(1024).decode())

client.close()
```

*At this stage, the project has progressed from basic TCP communication to manually handling the structure of HTTP requests. The next stage will build on this foundation by introducing HTTPS and encrypted communication.*

---

## How to run the HTTP server

---

*The HTTP server and client are currently designed to be run as two separate Python programs. The server must be started first so that it can begin listening for incoming connections.*

*To run the project:*

1. *Start an IDE of choice for python

2. *Run the HTTP server program.*

3. *Leave the server running while it waits for a client connection.*

4. *Run the HTTP client program separately to the HTTP server.*

5. *The client will connect to `127.0.0.1` on port `8000` and send the HTTP request.*

6. *The server will receive the request, parse the request line and headers, process the body if one is present, and display the resulting request data.*

*The current implementation is intended for local development and testing, with the server bound to `127.0.0.1:8000`. This keeps the testing environment local while the HTTP functionality is being developed.*

*Once the HTTP stage has been fully tested, the project will move on to HTTPS, where the connection will be protected using TLS encryption.*
