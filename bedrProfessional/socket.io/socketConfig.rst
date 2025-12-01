Socket.io is a js library for building the realtime, event-driven web applications. It enables bi-directional, event-driven communication between clients and server using the web sockets and fallback technologies
How it works ?
Socket.IO operates on top of HTTP and WebSocket protocols to provide:

1. Real Time Communication - low latency messaging
2. Fallback mechanism - fallbacks to long polling if the websocket disconnects
3. Automatic reconnection
4. Event-driven architecture
5. Built-in support for multiplexing and broadcasts

Socket.IO allows bi-directional communication between client and server. Bi-directional communications are enabled when a client has Socket.IO in the browser, and a server has also integrated the Socket.IO package. While data can be sent in a number of forms, JSON is the simplest.

This code defines a Socket class for initializing and managing a WebSocket connection using socket.io-client in a React Native application. It authenticates the socket connection using a token, and listens for various socket events such as connection, disconnection, errors, reconnection attempts etc.

(a) io: Socket.IO client used to establish real-time bidirectional communication.

           ┌────────────────────────────┐
           │  Start initialize(token)   │
           └────────────┬───────────────┘
                        │
                        ▼
      ┌────────────────────────────────────┐
      │   Is token provided?               │
      └────────────┬───────────────────────┘
                   │Yes                    │No
                   ▼                       ▼
        ┌────────────────────┐   ┌───────────────────────────────┐
        │ Connect to socket  │   │ Log error: "No token found"   │
        │ using token        │   └──────────────┬────────────────┘
        └────────────┬───────┘                  ▼
                     ▼                 ┌────────────────┐
            ┌────────────────┐         │   Exit method  │
            │ Listen to:     │         └────────────────┘
            │ - connect      │
            │ - disconnect   │
            │ - error        │
            │ - connect_err  │
            │ - ping         │
            │ - reconnect    │
            │ - reconnect_attempt │
            │ - reconnect_failed │
            └───────┬────────┘
                    ▼
        ┌────────────────────────────┐
        │  Log events accordingly    │
        └────────────┬───────────────┘
                     ▼
             ┌─────────────────┐
             │ Connection ready│
             └─────────────────┘


Advantages of This Pattern

Encapsulation: Socket logic is inside a single class.

Reusability: Singleton instance allows global access.

Debugging: Extensive event logging helps troubleshoot connection issues.

Authentication: Secure token-based connection setup.

🔒 Best Practices (Recommendations)

Handle token expiry: Add logic to refresh tokens if expired.

Retry strategy: Customize retry delays and max attempts.

Emit/Listen abstraction: Wrap emit() and on() methods in the class.

Disconnect method: Add disconnect() to cleanly close connections.

The class listens to several events for diagnostics and behavior tracking:

Event	Description
connect -->	Socket successfully connected.
disconnect -->	Socket disconnected from server.
error -->	A general error occurred.
connect_error -->	Error during the connection process.
ping -->	Server is checking connection health.
reconnect -->	Reconnected to the server.
reconnect_attempt -->	Attempting to reconnect.
reconnect_failed -->	Failed to reconnect.