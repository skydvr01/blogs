Augmented Reality is a rapidly developing technology in the gaming world. Augmented Reality, also known as AR, combines a game's visual and audio content with the user's surroundings in real-time, allowing players to use their existing environment and create a playing field within it. Proposed in the 1990s, AR was first utililized by Professor Tom Caudell of Boeing Computer Services in Seattle, USA. Since then, AR has grown in many different areas, especially since the introduction of smartphones around 2008.

Thanks to smartphones, Augmented Reality's outlook has changed significantly and is now easily available to the public. AR is quickly becoming part of our everyday lives. Several notable gaming, technology, and entertainment companies are developing further advancements in this exciting technology. This blog focuses on turning an iOS Augmented Reality peer-to-peer app into a client-server app. Future updates may include a guide to modifying Unity and Android apps to the client-server model.

AR Multiplayer games require low latency to simulate real life. If one device does something, all other devices connected to that game should see it in real-time. Apple's example AR Game (SwiftShot) uses the MultipeerConnectivity framework to join and send data between players. This peer-to-peer framework works well for a few devices, all near (close enough to utilize ad-hoc peer-to-peer WiFi or all on the same local WiFi network). Notwithstanding, the framework breaks down when more than 7 or 8 devices are connected. 

### Why Client-Server?
<add image>

Peer-to-Peer models require O(n2) connections, while Client-Server models only require O(n) connections. Therefore, Client-Server models can support many more devices running the same app and can quickly deliver data to each device.

The main advantage of the MultipeerConnectivity framework is that it allows devices to "network" without being on WiFi (each iOS device acts as a transmitter, receiver, and router). However, Edge Cloudlet servers can be connected via cellular data as well.

Note: When we convert this app to Client-Server, it makes sense to put our server on an Edge Cloudlet, so that we can reduce latency further. An edge server can also offload game synchronization work, store point cloud maps (to enable app persistence), and divide users into different “rooms” to easily keep track of several game states.

### Websockets and Socket.IO:
There are a couple different ways to implement a Client-Server model. We can use REST calls, but as explained in Computer Vision Persistent TCP Connection, REST calls are not very efficient, especially for an AR Game (you have open up a new HTTP connection every time a device needs to communicate).

Websockets provides a full-duplex TCP connection for real-time communication. The Websocket Protocol starts with an HTTP handshake and then makes use of a TCP connection. Both server and clients push data asynchronously to the other and utilize callbacks to handle incoming data. Websockets are persistent and enable real-time communication.

Socket.IO is a Javascript library that uses the Websocket Protocol. Javascript also has a Websockets library, but Swift did not have a compatible client side Websockets library for this game. (The Unity Ping Pong Game uses Websockets instead of Socket.IO). The Socket.IO library provides some features on top of the Websocket protocol. This library includes "rooms" to easily divide different clients into different games,  allow more flexibility in sending multiple messages/data at a time, and furnishes the ability to add metadata to packets to quickly implement callbacks/event listeners.

Below is example skeleton code that implements Socket.IO callbacks on server-side and client-side. The server is Node.js and the client is Swift.

### Node.js Server Code Skeleton:
Socket.IO has server events and socket events. Most of the work is in implementing the event listeners for each event. The server event: 'connection' occurs when an incoming client connection (the parameter: socket) is successful. In that event listener, we implement the socket event callbacks. These are called when specific events come in from that particular client.

Socket.IO reserves some events such as ‘error’ and ‘disconnect’ which are triggered automatically with an error and socket disconnect, but the rest of the events are determined by the developer (’login', ‘bullet’, ‘worldMap’, ‘score’).

Other functions to notice are the .join() and .emit() functions. Socket.IO provides an easy way to separate different connections into different “rooms”. In this game, we separated different clients into different games based on the gameID using the .join() function. You can easily send data to only clients in that “room”. The .emit() function sends data to the specified clients.

The socket can also be assigned metadata (ie. socket.gameID = gameID), which makes it easy to keep track of sockets connecting and disconnecting (although Socket.IO handles most of that for you too). Here, socket refers to the client socket.
```
var io = require('socket.io')(1337) // Listen on port 1337

io.on('connection', function(socket) { 

    socket.on("login", function(gameID, username) {
        socket.gameID = gameID;   // easily assign meta data to socket
        socket.username = username;
        socket.join(gameID);
        console.log("gamid: " + gameID + ", username " + username);
    });

    socket.on("bullet", function(gameID, bullet) {
        socket.to(gameID).emit("bullet", bullet);
    });

    socket.on("worldMap", function(gameID, worldMap) {
        socket.to(gameID).emit("worldMap", worldMap);
    });

    socket.on("score", function(gameID, username) {
        console.log("score");
    });

    socket.on("error", function(err) {
        console.log("Server socket error: ");
        console.log(err.stack);
    });
    
    socket.on('disconnect', function(reason) {
        console.log(socket.username + “ in game:” + socket.gameID + “ disconnected”);
        console.log(reason);
    });
});
```

### Swift Client Code Skeleton:
iOS' ARKit library does most of the heavy lifting in an AR game. The library generates a point cloud to map the area, sets up anchors and feature points to track movement, and creates and renders augmented reality objects. All we have to do is send the data generated by ARKit to other devices.

Instead of using MultipeerConnectivity to send data, you can use the Socket.IO-Swift-Client library to communicate with the Socket.IO.Node.js server.

To do this, we have first to create a connection with our server. Given the name of the server or IP address and the port to connect to, we can create our TCP connection. Here, socket refers to the server socket.
```
let manager = SocketManager(socketURL: URL(string: "wss://host:port/")!)
socket = manager!.defaultSocket
setUpSocketCallbacks() // custom function to set up socket callbacks
socket.connect() // connect to server socket
```

Just like in the server code, we use the .emit() function to send the server data, where it is forwarded to the other clients connected to the server/game. For example, the following code snippet converts an ARAnchor to binary data and then sends that data to the Node.js server.
```
guard let data = try? NSKeyedArchiver.archivedData(withRootObject: anchor, requiringSecureCoding: true) else{fatalError("can't encode anchor")}
socket.emit("bullet", gameID!, data)
```
The majority of the work is in implementing callbacks for when the client receives data from the server.

Data comes in as an array and you also can customize acknowledgements. Notice that the .emit() function takes in several parameters. The first is the “event”, and anything following that is data that you can send to the server. Note: the acknowledgement is a callback function that can be implemented as the last parameter in the .emit() function.
```
func setUpSocketCallbacks() {

    socket.on(clientEvent: .connect) { data, ack in
        print("Socket connected")
        self.socket.emit("login", self.gameID!, self.userName!)
    }
        
    socket.on("bullet") { [weak self] data, ack in
        guard let bulletData = data[0] as? Data else { return }
        print("received bullet")
    }
                
    socket.on("worldMap") { [weak self] data , ack in
        guard let worldData = data[0] as? Data else { return }
        print("received worldMap")
    }
        
    socket.on("otherUsers") { [weak self] data, ack in
        guard let otherUsers = data[0] as? NSDictionary else { return }
        print("received otherUsers")
    }
}
```
Note: The above server and client code does nothing more than send data between client, server, and other clients. There is no game synchronization implemented in the server code.

 
