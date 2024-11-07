# simple-tcp-chat

# Overview
This project is a simple client-server chat application implemented in Python using the socket and threading libraries. It enables multiple clients to connect to a server and exchange messages in a group chat environment. The server handles message broadcasting, and each client has its own username.

# Requirements
Python 3.x
Basic knowledge of socket programming and threading

# How It Works
The server listens for incoming connections, accepts them, and handles each client in a separate thread. When a client sends a message, the server broadcasts it to all other connected clients. Clients can enter their username and send messages, which are displayed to all users in the chat.

# Project Structure
1. Server Code: Manages client connections, broadcasts messages, and removes clients on disconnection.
2. Client Code: Connects to the server, receives messages, and allows the user to send messages.

# Server Code
The server code is responsible for:

1. Setting up the server to listen on a specified host and port.
2. Accepting client connections and storing each client socket and username.
3. Broadcasting messages to all connected clients except the sender.
4. Removing clients from the chat when they disconnect.

Server Code Example

import socket
import threading

# Server setup
host = '127.0.0.1'  # Localhost
port = 12345
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind((host, port))
server_socket.listen()
clients = []
usernames = []

# Broadcast message to all clients
def broadcast(message, client_socket=None):
    for client in clients:
        if client != client_socket:
            client.send(message)

# Handle individual client connection
def handle_client(client_socket):
    while True:
        try:
            message = client_socket.recv(1024)
            broadcast(message, client_socket)
        except:
            index = clients.index(client_socket)
            clients.remove(client_socket)
            client_socket.close()
            username = usernames[index]
            broadcast(f'{username} has left the chat.'.encode('utf-8'))
            usernames.remove(username)
            break

# Accept connections
def receive_connections():
    print("Server is running and listening...")
    while True:
        client_socket, client_address = server_socket.accept()
        print(f"Connection from {client_address} established.")
        client_socket.send("USERNAME".encode('utf-8'))
        username = client_socket.recv(1024).decode('utf-8')
        usernames.append(username)
        clients.append(client_socket)
        print(f"Username of the client is {username}")
        broadcast(f"{username} has joined the chat!".encode('utf-8'))
        client_socket.send("Connected to the server!".encode('utf-8'))
        thread = threading.Thread(target=handle_client, args=(client_socket,))
        thread.start()

receive_connections()


# Client Code
The client code is responsible for:

1. Connecting to the server using the provided host and port.
2. Allowing the user to choose a username.
3. Starting two threads: one for receiving messages from the server and another for sending messages.
4. Handling message input and allowing users to exit the chat by typing "quit."

# Client Code Example

import socket
import threading

# Setup for the client
host = input("Enter the server IP address (default is '127.0.0.1' for localhost): ") or '127.0.0.1'
port = int(input("Enter the server port (default is 12345): ") or 12345)
client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    client_socket.connect((host, port))
    print("Connected to the server successfully!")
except Exception as e:
    print(f"Connection failed: {e}")
    exit(1)

# Function to receive messages
def receive_messages():
    while True:
        try:
            message = client_socket.recv(1024).decode('utf-8')
            if message == 'USERNAME':
                client_socket.send(username.encode('utf-8'))
            else:
                print(message)
        except:
            print("An error occurred while receiving a message.")
            client_socket.close()
            break

# Function to send messages
def send_messages():
    while True:
        message = input("")
        if message.lower() == 'quit':
            client_socket.send(f"{username} has left the chat.".encode('utf-8'))
            client_socket.close()
            print("You left the chat.")
            break
        else:
            client_socket.send(f'{username}: {message}'.encode('utf-8'))

# Get username and start threads
username = input("Choose a username: ")
client_socket.send(username.encode('utf-8'))
print("Type 'quit' to exit the chat.")

# Start the receive and send message threads
receive_thread = threading.Thread(target=receive_messages)
receive_thread.start()
send_thread = threading.Thread(target=send_messages)
send_thread.start()


# Running the Application
1. Start the Server: Run the server code on the host machine. It will begin listening for connections.
2. Connect Clients: Run the client code on one or more machines, entering the server's IP address and port. Each client should choose a username.
3. Chat: Clients can send messages, and they will be broadcast to all connected users. To leave, type quit.


# Notes
1. Ensure that the server is running before starting any clients.
2. Both server and client should use the same IP address and port for successful connection.
3. Typing quit in the client console disconnects that user.

This setup provides a simple way to implement a multi-client chat room in Python.
