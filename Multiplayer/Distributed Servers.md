[Home](../README.md) > [Multiplayer](README.md) > [Distributed Servers](Distributed%20Servers.md)
# Distributed Servers
## References
- https://robertsspaceindustries.com/en/comm-link/transmission/18397-Server-Meshing-And-Persistent-Streaming-Q-A
- https://github.com/channeldorg/channeld
- https://github.com/kbengine/kbengine
- https://github.com/ketoo/NoahGameFrame
- https://github.com/hagukin/Rovenhell_UE

## Architecture decisions
### Master-slave servers
A master-slave architecture distributes server responsibilities to improve performance, scalability, and fault tolerance.

- Master Server → Central authority that handles global game state, matchmaking, player authentication, and database writes. It does not process game logic for individual matches.
- Slave Servers (Game Servers) → Run individual game instances, handling physics, player interactions, and other game logic. They report results to the master server but operate independently per match.

In a master-slave distributed server setup, where there are multiple game servers, the key idea is that the master server assigns the client to a specific game server for the duration of their gameplay session. After this assignment, the client only communicates directly with the assigned game server for the gameplay, not the master server anymore.

### Web service as bridge
The web service here facilitates communication between clients and the dedicated Unreal servers in a distributed game session. Once the web service has identified the relevant game server for the player’s input, it forwards the input to that dedicated Unreal server, ensuring the server with the authority over the player’s state processes the input. The web service might also relay certain replicated data (game state changes) to clients or other servers in the distributed architecture. 

**Cross-server synchronization**: If players are interacting with different servers (e.g., moving between areas), the web service ensures state consistency.

**State updates:** The game servers send the latest player state updates back to the web service, which then pushes them to the relevant clients.

## Server-to-server communication
Server-to-server communication cannot be done via standard network usages in unreal. We can use ISocketSubsystem to establish an open socket connection between both server and transfer data this way.

1) Establish Ownership of Actors. Which server has authority over which object.
2) Replicate owned actors to other servers and listen to remote actor data.
3) Consider using other servers as 'thread' by sending inputs and receiving outputs for heavy calculations when the other server has all the data needed.


Creating Sockets for server to server communication:
```cpp
// Create a socket
FSocket* Socket = ISocketSubsystem::Get(PLATFORM_SOCKETSUBSYSTEM)->CreateSocket(NAME_Stream, TEXT("ServerSocket"), false);

// Set up address
FIPv4Address Addr;
FIPv4Address::Parse(TEXT("127.0.0.1"), Addr);
TSharedRef<FInternetAddr> InternetAddr = ISocketSubsystem::Get(PLATFORM_SOCKETSUBSYSTEM)->CreateInternetAddr();
InternetAddr->SetIp(Addr.Value);
InternetAddr->SetPort(7777);

// Connect the socket
if (Socket->Connect(*InternetAddr))
{
    // Send data
    const char* Message = "Hello Server";
    int32 BytesSent;
    Socket->Send((uint8*)Message, strlen(Message), BytesSent);
}

// Bind and listen
if (Socket->Bind(*InternetAddr) && Socket->Listen(8)) // Max 8 pending connections
{
    UE_LOG(LogTemp, Log, TEXT("Server listening on port 7777..."));
        
    // Accept loop (implement this on a thread)
    FSocket* ClientSocket = Socket->Accept(TEXT("ClientSocket"));
    if (ClientSocket)
    {
        UE_LOG(LogTemp, Log, TEXT("Client Connected!"));

        // Receive data buffer
        uint8 Data[1024];
        int32 BytesRead;

        // Blocking receive (should be in a loop for continuous reading)
        if (ClientSocket->Recv(Data, sizeof(Data), BytesRead))
        {
            FString ReceivedMessage = FString(ANSI_TO_TCHAR(reinterpret_cast<const char*>(Data)));
            UE_LOG(LogTemp, Log, TEXT("Received: %s"), *ReceivedMessage);
        }

        // Add received message to a TQueue for game thread to read.
    }
}

// Clean up
Socket->Close();
ISocketSubsystem::Get(PLATFORM_SOCKETSUBSYSTEM)->DestroySocket(Socket);
```

© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). 