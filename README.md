# socket_programming
send and received data from python 
Socket Types
Depending on the transport layer protocol used, sockets can either be:
SOCK_DGRAM for message-oriented datagram transport. These sockets are usually associated with the User Datagram Protocol (UDP) which provides an unreliable delivery of individual messages. Datagram sockets are commonly used when the order of messages is not important, such as when sending out the same data to multiple clients.
SOCK_STREAM for stream-oriented transport often associated with the Transmission Control Protocol (TCP). TCP provides a reliable and ordered delivery of byte between two host, with error handling and control, making it useful for implementing applications that involve transfer of large amounts of data.




An Overview of The Sockets Module
To use sockets in Python we will need to import the socket module.
>>> import socket
The primary way of using the sockets module is through the sockets() function which returns a socket object with methods to implement various system socket calls. Python sockets supports a number of address families under the network layer protocol, mainly:
AF_INET — this is the most common, and uses IPv4 for network addressing. Most of the internet networking is presently done using IPv4.
AF_INET6 — this is the next generation of the internet protocol using IPv6 and provides a number of features not available under IPv4.
AF_UNIX — finally, this is the address family for Unix Domain Sockets (UDS), an inter-process communication protocol available on POSIX-compliant systems. This implementation allows passing of data between processes on an operating system without going through the network.
Network related services
In this section we will look at some socket functions and methods that provide access to network related tasks.
gethostname() to get the official name of the current host.
>>> import socket
>>> print(socket.gethostname())
rodgers-PC
gethostbyname() converts the name of a server into its numerical address by consulting the operating system’s DNS configuration.
>>> import socket
>>> print(socket.gethostbyname("google.com"))
216.58.223.78
gethostbyname_ex() to get more naming information about a server.
>>> import socket
>>> name, aliases, addresses = socket.gethostbyname_ex("google.com")
>>> print("Name: ",name)
Name:  google.com
>>> print("Aliases: ",aliases)
Aliases:  []
>>> print("Addresses: ",addresses)
Addresses:  ['216.58.223.78']
gethostbyaddr() to perform a reverse lookup for a domain’s name.
getfqdn() to convert a partial domain into a fully qualified domain name.
More network-related functions can be accessed from the python documentation referenced in the conclusion section below.
TCP/IP client-server communication
As we have established, sockets can either be configured to act as a server or client, to achieve bi-directional communication over TCP using the SOCK_STREAM family. In this example, we shall implement a simple echo application that receives all incoming data and sends them back to the sender. For that we will implement both client and server sockets. Furthermore, we will use the local loopback address 127.0.0.1 or localhost for our connections.
Echo Server
To set up a server, it must perform the sequence of methods socket(), bind(), listen(), and accept() .
socket() creates a new socket given the address family and a socket type.
bind() binds our socket object to a particular address composed of a host and port number.
listen() allows the server to start accepting connections and takes in an argument, backlog, which is the number of unaccepted connections that the system can allow before refusing all new connections.
accept() accepts incoming connections and returns a a tuple of (conn, address) where conn is a new socket that can be used to send and receive messages from the connection and address is the address bound to the socket on the other end of the connection.
close() marks the socket as closed and can no longer accept connections.

Server implementation of TCP/IP socket
We initialize a socket object, sock by passing in the address family ( socket.AF_INET) and socket type ( socket.SOCK_STREAM) to the socket.socket() function.
Next, we bind the socket object to an address of the form ('localhost',10000) — bind to localhost on port 10000 , using the bind() method.
Listen for incoming connections with a backlog of 1.
Continuously wait for and accept connections by calling sock.accept() and unpacking the return value into connection and client_address .
Call recv(16) on the returned connection to receive the data on chunks of 16.
If data as been received, then we transmit the received data back to the sender by calling the method sendall(data) on the connection, otherwise we print out a statement indicating no data has been received.
Finally when communication with the client is complete — all the chunks of the message have been transmitted, we call close() on the connection object. We use a try/finally block to ensure that close() is called even in the event of an error when transmitting messages.
Echo Client
Unlike a server, a client only needs to execute the sequence of socket() and connect().
connect() connects the socket to an address.

Client implementation of TCP/IP socket
In this example:
We initialize a socket object as in the server.
Next we connect the socket to the same address that the server is listening on, in this case, ('localhost',10000) , using the connect(address) method.
In a try/finally block, we compose our message as a byte string and use sendall() method on the socket object with the message as an argument.
We then set up variables amount_received with an initial value of 0 and amount_expected which is just the length of our message, to keep track of the message chunks as we receive them.
Calling recv(16) on the socket object allows us to receive the message from our server in chunks of 16, and we keep receiving until amount_received is equal to amount_expected .
Finally we mark the socket as closed.
Running out server and client scripts on separate terminal windows, this is the output from the server;
$ python3 echo_server.py                                                                                            Starting up on localhost port 10000
waiting for a connection
connection from ('127.0.0.1', 49964)
received b'This is our mess'
sending data back to the client
received b'age. It is very '
sending data back to the client
received b'long but will on'
sending data back to the client
received b'ly be transmitte'
sending data back to the client
received b'd in chunks of 1'
sending data back to the client
received b'6 at a time'
sending data back to the client
received b''
no data from ('127.0.0.1', 49964)
Closing current connection
waiting for a connection
And the client;
$ python3 echo_client.py                                                                                             connecting to localhost port 10000
sending b'This is our message. It is very long but will only be transmitted in chunks of 16 at a time'
received b'This is our mess'
received b'age. It is very '
received b'long but will on'
received b'ly be transmitte'
received b'd in chunks of 1'
received b'6 at a time'
closing socket
UDP client-server communication
Unlike the case with TCP transmission which streams messages in an ordered manner, UDP is message oriented and does not require a long-lived connection. A message in this case has to fit within a single datagram and delivery is not assured.
Echo Server
Here, we’ll only execute the socket() and bind() sequence since, there isn’t really a connection to listen for. Instead we only need to bind the socket to a particular address and wait for incoming messages. We will then read the incoming messages using the recvfrom() method and send them back with sendto().
recvfrom() receives data from a socket and return a tuple of (bytes, address) where bytes is a bytes object containing the received data and address is the address of the sender.
sendto(bytes,address) sends data(given by bytes) to a socket bound to the address as defined by address .

Server implementation of UDP socket
In the example above:
We create a socket object using socket.socket(socket.AF_INET,socket.SOCK_DGRAM) . Please note that here we use the socket.socket_DGRAM socket type since we are using UDP.
Next we bind the socket to the ('localhost',10000) and wait for incoming messages.
When a message arrives, we proceed to read it with recvfrom(4096) , where 4096 is the number of bytes to be read, and unpack the return value in to data and address .At this point we can print out the length of data.
If some data has been received, we send it back to the sender using the sendto() method and print out the length of the return value — which is the sent data.
Echo Client
This client is similar to the server above, only that it doesn’t bind the socket to any address. Instead, the it uses sendto() to send messages to the server’s address.

Client implementation of UDP socket
In the script:
We instantiate a sock object as in the server above.
We then compose a message as a byte string, and define a server_address as a tuple of the host and the port number bound to the server we wish to send messages to.
Inside a try/finally block, we send the message and wait for a response, printing the the data in both cases.
Finally we mark the socket as closed.
This is the output when the client and server scripts are run on the server:
$ python3 echo_server_udp.py                                                                                         starting up on localhost port 10000
waiting to receive message
received 48 bytes from ('127.0.0.1', 34351)
b'This is our message. It will be sent all at once'
sent 48 bytes back to ('127.0.0.1', 34351)
waiting to receive message
And the client:
$ python3 echo_client_udp.py                                                                                         sending b'This is our message. It will be sent all at once'
waiting to receive
received b'This is our message. It will be sent all at once'
closing socket
Unix Domain Sockets
These are largely similar to TCP sockets with basically two exceptions:
The socket address in this case is a path on the file system such as ./socket_file unlike in the TCP sockets where the address was a tuple of a host name and a port number.
Since the node created to represent the socket is a file, it persists even after the socket is closed, and as such it’s supposed to be removed whenever the server starts up.
To implement a similar client-server communication setup with UDS, we would need to slightly modify the example of TCP above.
Echo Server

Server implementation of Unix socket
Here we alter the server_address variable in the script to a file system path, in this case ./socket_file .
Since we also need to make sure the node doesn’t already exist when starting the server, we use a try/except block to delete the file using os.unlink() if it exists.
Echo Client

Client implementation of Unix socket
In this case, we also replace the server_address variable to the file path that has been bound to the server.
Running the client and server results in an output almost similar to the TCP case. For the server we get:
$ python3 echo_server_uds.py                                                                                         Starting up on ./socket_file
waiting for a connection
connection from 
received b'This is our mess'
sending data back to the client
received b'age. It is very '
sending data back to the client
received b'long but will on'
sending data back to the client
received b'ly be transmitte'
sending data back to the client
received b'd in chunks of 1'
sending data back to the client
received b'6 at a time'
sending data back to the client
received b''
no data from 
Closing current connection
waiting for a connection
And the client:
$ python3 echo_client.uds.py                                                                                         connecting to ./socket_file
sending b'This is our message. It is very long but will only be transmitted in chunks of 16 at a time'
received b'This is our mess'
received b'age. It is very '
received b'long but will on'
received b'ly be transmitte'
received b'd in chunks of 1'
received b'6 at a time'
closing socket
Permissions
Since UDS sockets are represented by nodes on the the file system, this implies that standard file system permissions can be used to control access to the server. For instance, lets try to change the ownership of the existing node to a root user.
$ ls -la ./socket_file
srwxr-xr-x 1 rodgers rodgers 0 Nov  1 15:24 ./socket_file
$ sudo chown root ./socket_file
$ ls -la ./socket_file                                                                                               srwxr-xr-x 1 root rodgers 0 Nov  1 15:24 ./socket_file
$ python3 echo_client.uds.py
connecting to ./socket_file
[Errno 13] Permission denied
As we can see, connecting to the server as a regular user fails, meaning only a user with the correct permissions (in this case the root user), can access the server.
Dealing with Multiple connections — Multicast
When dealing with multiple clients, maintaining several point-to-point connections can be cumbersome for applications due to the increases bandwidth and processing needs. This is where multicast messages come in. With multicast, messages are delivered to multiple endpoints simultaneously. This method achieves increased efficiency since delivery of messages to all recipients is delegated to the network infrastructure.
Multicast messages are sent using UDP, since TCP assumes a pair of communicating endpoints. The addresses for multicast messages, identified as multicast groups, are a subset of IPv4 addresses, usually between the range of 224.0.0.0 to 239.255.255.255. Network routers and switches treat addresses in this range as special since they are reserved for multicast, ensuring messages sent to the group are distributed to all clients that join the group.
Send Multicast Messages
To send messages we use an ordinary sento() method with a multicast group as the address. Moreover, we also need to specify a Time To Live(TTL) value which determines how far the messages should be broadcast from the sender. The default TTL of 1, will result in messages being sent only to hosts within the local network. We shall use setsockopt() with the IP_MULTICAST_TTL option to set the TTL, which should be packed into a single byte.
We shall also set a timeout value on the socket, to prevent it from waiting indefinitely for responses, since we have no idea how many responses we expect to get from the network.

Multicast sender socket implementation
In this example:
We create a socket of type socket.SOCK_DGRAM , compose or message as a byte string and bind our socket to the multicast group address (‘224.10.10.10’, 10000) .
We then set a timeout of 0.2 using sock.settimeout(0.2) .
Using the struct module, we pack the number 1 into a byte and assign the byte to ttl .
Using setsockopt() we set the IP_MULTICAST_TTL option of the socket to the ttl we just created above and send the message using sendto() .
We then wait for responses from other hosts on the network, as long as the wait hasn’t timed out and print out the responses.
Receiving Multicast Messages
To receive messages, after creating our ordinary socket and binding it to a port, we would need to add it to the multicast group. This can be done by using the setsockopts() to set the IP_ADD_MEMBERSHIP option, which should be packed into an 8-byte representation of the multicast group, and the network interface on which the server should listen for connections. We shall use socket.inet_aton() to convert the multicast group IPv4 address from dotted-quad string format (‘224.10.10.10’) to 32-bit packed binary format.

Multicast receiver socket implementation
In this example:
To add the socket to the multicast group, we use struct to pack the group address given by socket.inet_aton(multicast_group) and the network interface given by socket.INADDR_ANY into an 8-byte representation, which we then set to the socket’s IP_ADD_MEMBERSHIP .
As the socket receives messages, we use recvfrom(1024) to unpack the response into data and address , and send out an acknowlegement to the address using sendto() .
On running both scripts, on different hosts (A and B), this is the output of the multicast sender on Host A:
[A]$ python3 multicast_sender.py                                                                                        sending b'very important data'
waiting to receive
received b'ack' from ('192.168.100.2', 10000)
waiting to receive
received b'ack' from ('192.168.100.13', 10000)
waiting to receive
timed out, no more responses
closing socket
And receiver on Host B:
[B]$ python3 multicast_receiver.py                                                                                      waiting to receive message
received 19 bytes from ('192.168.100.2', 48290)
b'very important data'
sending acknowledgement to ('192.168.100.2', 48290)
waiting to receive message
Sending Binary Data
So far we have been transmitting streams of text data encoded as bytes through our sockets. Sockets can also transmit binary data as streams of bytes. We can use struct to pack the binary data to prepare it for transmission. However, when sending multi-byte data, it is important that both sender and recipient know the order in which the bytes are. This is useful in reconstructing the bytes at the recipient’s end.
Binary Client
We shall now implement a binary client that sends an integer, a string and a floating point number by packaging the data into a series of bytes.

Implementation of a TCP/IP socket receiving binary data
Here we set up a socket of type socket.SOCK_STREAM and connect it to our server address. We then use a struct.Struct('I 2s f') specifier to pack the data before sending with sendall().
Binary Server
On the other hand we have a server, that listens on the server address for incoming connections. After accepting a connection and receiving data we then proceed to unpack the data using a struct.Struct('I 2s f') specifier. Note that we use the same specifier on both ends of the communication so that the received bytes are interpreted in the same order they were packed.

Implementation of a TCP/IP socket sending binary data
The output we get from the client:
$ python3 binary_client.py                                                                                           values = (1, b'ab', 2.7)
sending b'0100000061620000cdcc2c40'
closing socket
And the server:
$ python3 binary_serverr.py                                                                     waiting for a connection
received b'0100000061620000cdcc2c40'
unpacked: (1, b'ab', 2.700000047683716)
waiting for a connection
From the server output, the floating point numbers loses some precision as a result of packing and unpacking. Otherwise everything else is transmitted as is.
Blocking and Timeouts
By default, sockets operate in a blocking mode. This means that sending or receiving data pauses execution of the program until the the socket that’s receiving or sending data is ready. Sometimes, this mode of operation can result in a dreadlock whereby both endpoints are waiting for the other to send or receive data.
To try and overcome this challenge, sockets have an option to unset blocking, using the method socket.setblocking() , which has a default value of 1, i.e blocking is on. To disable blocking we call socket.setblocking(0) . However, this presents yet another challenge; if a socket is not ready for an operation with blocking disabled, then a socket.error is raised.
Another solution therefore, involves setting a timeout, using socket.settimeout(float) to a number(float) of seconds during which blocking is on before evaluating whether the socket is ready or not. This timeout method is applicable in most cases since it acts as a compromise between blocking and not blocking.
Troubleshooting
As we can all attest to, some times things don’t work and more often than not, it’s never obvious why. It could be a issue with the code in which case looking at the errors and traceback could point us in the right direction. In such cases, referring to the official documentation would be the first option.
However, sometimes it could be configuration troubles on the client, server, or even the network infrastructure. To probe such issues we could use ping or netstat to identify the source of problems.
ping — works directly with the TCP/IP stack, independent of any other application running on a host, to determine the status of the host, whether it’s up or not. ping works by sending ICMP echo request packets to the target host and waiting for an ICMP echo reply.
Below is an example of ping running on Ubuntu:
$ ping -c 4 127.0.0.1                                                                                                PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.055 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.067 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.077 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.067 ms
--- 127.0.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3057ms
rtt min/avg/max/mdev = 0.055/0.066/0.077/0.011 ms
netstat —gives information sockets and their current states. Let’s fire up the TCP echo server and observe the output of netstat . From this we can tell that our server is presently using thetcp protocol, listening (state of LISTEN) on port 10000 and host 127.0.0.1 .
$ netstat -an                                                                                                        Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address     Foreign Address         State tcp        0      0 127.0.0.1:3306    0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:10000   0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.53:53     0.0.0.0:*               LISTEN
Conclusion
We have covered quite a number of concepts in this article. However sockets and networking are very broad and it is not possible to cover everything in a single article. To have a better understanding of these concepts, it is imperative that we practise more. The socket module in particular has so many methods, functions and attributes that we haven’t covered, please feel free to refer to the official documentation. The internet also also a lot of information on socket programming and networking in general.
