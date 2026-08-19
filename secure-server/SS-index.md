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
